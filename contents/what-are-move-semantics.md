# 移动语义的意思？

## 简短回答

结合示例代码是理解移动语义最简单的方法。让我们考虑一个最简单的 string 类，仅仅包含一个指向堆内存地址的指针：

```C++
#include <cstring>
#include <algorithm>

class string
{
    char* data;

public:

    string(const char* p)
    {
        size_t size = strlen(p) + 1;
        data = new char[size];
        memcpy(data, p, size);
    }
```

既然我们要手动管理资源，那么就要遵循 [rule of three](http://en.wikipedia.org/wiki/Rule_of_three_%28C++_programming%29) 原则。我们先编写析构函数和拷贝构造函数，后续在编写赋值操作符。

The copy constructor defines what it means to copy string objects. The parameter const string& that binds to all expressions of type string which allows you to make copies in the following examples:

拷贝构造函数定义了 string 对象的拷贝行为。参数 const string& 可以绑定到如下 string 类型的表达式上 从而允许你对 string 对象进行拷贝。

```C++
string a(x);                                    // Line 1
string b(x + y);                                // Line 2
string c(some_function_returning_a_string());   // Line 3
```

现在到了理解移动语义的关键点了。你有没有注意上上述三条语句，只有第一条对 x 的复制必须要执行 深拷贝 ？因为我们后续可能还会用到 x ，如果 x 在此处发生变动，是不符合我们的预期的。你有诶有注意到我说了三次 x 每次都是指的同一个对象？我们把 类似 x 的表达式称为 左值。

第二行第三行的参数就不是左值而是右值。因为底层的 string 对象是没有名字的，所以后续代码就没有办法再去访问 这个 string 对象的内容了。右值是指在下一个分号处就销毁的临时对象（确切说是在字面上包含这个临时对象的完整表达式结束的时候）。这很重要，因为在 b 和 c 的初始化过程中，我们可以使用源字符串做任何我们想要做的事情，而对后续代码没有任何影响。（This is important because during the initialization of b and c, we could do whatever we wanted with the source string, and the client couldn't tell a difference）

C++0X 引入了 右值引用 机制，使用右值引用作为参数可以让我们以函数重载的方法区分出来实参是右值的调用。我们只需要写一个形参是右值引用的构造函数，在这个构造函数里面我们可以对右值做任何事情，只要最后把它置于一个有效状态就可以了（可以正常被析构的状态）。

```C++
string(string&& that)   // string&& is an rvalue reference to a string
{
    data = that.data;
    that.data = nullptr;
}
```

这段代码并没有进行整个字符串的复制而仅仅复制了指针然后把源指针设置为 nullptr。从结果上来说，我们 “偷取” 了源字符串的内容。在强调一次，对后续代码来说任何情况下都无法得知我们修改了参数字符串的状态。既然我们并没有进行实质的资源拷贝操作，我们称这个为 移动构造函数。它的作用是把资源从一个对象移动到另一个对象中。

恭喜你，你已经明白移动语义的基本知识。下面我们继续实现赋值操作符，如果你还不了解 [copy-and-swap](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom) 惯用法，先去学习一下，然后再继续看下面的内容。

```C++
    string& operator=(string that)
    {
        std::swap(data, that.data);
        return *this;
    }
};
```

仅此而已？右值引用在哪儿？我的回答是：在这儿我们并不需要。

请注意，参数 that 是传值的方式。所以 实参 that 会像其他 string 对象一样被正常初始化。确切的说在 C++98 的年代，是通过拷贝构造函数初始化的，C++0x 之后， 编译器会根据传递的实参的类型来决定是调用拷贝构造函数还是移动构造函数。

所以，如果代码是 a = b，那么因为 b 是左值，编译器会通过调用拷贝构造函数来初始化 that 。然后赋值操作符会把 a 和这个临时创建的深拷贝的 that 对象进行交换。这就是 copy-and-swap 惯用法的精髓-- 拷贝对象，交换新建对象的内容，然后在离开作用域时自动销毁新对象。并没有什么新花样。

但是如果代码是 a = x + y， that 就会被移动构造函数创建（以为 表达式 x + y 是右值）。仅需移动操作，无需进行深拷贝。that 仍然是一个独立的对象，只是他的创建并不是通过复制堆数据实现的，仅仅进行了移动。因为 x + y 是右值，所以并不需要拷贝，再说一遍：对右值对象进行移动操作是没问题的。

总结一下， 拷贝构造函数是对源对象的深拷贝，因为我们要保证源对象不能被改变。与之相反，移动构造函数则可以仅仅复制指针，然后把源对象的指针设置为 nullptr。 在这儿简单的把源对象指针置空是可行的，因为后续代码无法在访问这个对象了。

我希望这个答案表达清楚了移动语义的主要概念。为了保持答案的简短易懂，我故意忽略了大量关于右值引用和移动语义的细节内容。如果你想深入了解可以继续阅读后续内容。

## 深入回答

### 介绍

移动语义 允许一个对象在特定的情况下可以直接获取另一个对象所管理外部资源的所有权，该语义的重要性展现在下面两个方面

1. 用开销小的移动操作代替开销很大的复制操作。可以参考上面答案的例子。注意如果一个对象并不直接或者间接（通过成员变量）的管理外部资源，那么移动语义相对于拷贝语义并没有优势，此种情况下移动和复制实质上是一样的

```C++
class cannot_benefit_from_move_semantics
{
    int a;        // moving an int means copying an int
    float b;      // moving a float means copying a float
    double c;     // moving a double means copying a double
    char d[64];   // moving a char array means copying a char array

    // ...
};
```

2. 安全的实现仅支持移动的类型。意即实现一些 拷贝 没有意义，但是移动却有意义的类型。比如 锁，文件句柄以及唯一所有权的智能指针。注意本答案会讨论在 C++11 中被 std::unique_ptr 替换的 C++98 标准库中的 std::auto_ptr。有经验的 C++ 程序员多少对 std::auto_ptr 有所了解，可以通过讨论它所展现出来的 “移动语义“ 来展开 C++11 移动语义的讨论。

### 什么是移动？

C++98 标准库提供了一个具有单一所有权语义的智能指针类型 std::auto_ptr<T>, 使用它可以确保即使在发生异常的情况下也可以保证动态分配的对象被正确的释放。

```C++
{
    std::auto_ptr<Shape> a(new Triangle);
    // ...
    // arbitrary code, could throw exceptions
    // ...
}   // <--- when a goes out of scope, the triangle is deleted automatically
```

auto_ptr 有一个反常识（unusual）的 拷贝 行为

```C++
auto_ptr<Shape> a(new Triangle);

      +---------------+
      | triangle data |
      +---------------+
        ^
        |
        |
        |
  +-----|---+
  |   +-|-+ |
a | p | | | |
  |   +---+ |
  +---------+

auto_ptr<Shape> b(a);

      +---------------+
      | triangle data |
      +---------------+
        ^
        |
        +----------------------+
                               |
  +---------+            +-----|---+
  |   +---+ |            |   +-|-+ |
a | p |   | |          b | p | | | |
  |   +---+ |            |   +---+ |
  +---------+            +---------+
```

注意： 通过 a 创建 b 并没有复制 triangle， 而仅仅是把 triangle 的所有权传递给了 b 。我们仍然称他为 从 a 移动到 b，或者 trangle 从 a 移动到了 b。这听起来很奇怪，因为 trangle 数据本身在内存中的位置并没有发生变化。

>移动一个对象意味着把他管理的资源的所有权移交给另一个对象。

auto_ptr 的构造函数看起来是执行了下述操作（简化的示例代码）

```C++
auto_ptr(auto_ptr& source)   // note the missing const
{
    p = source.p;
    source.p = 0;   // now the source no longer owns the object
}
```

### 移动的用法和问题

auto_ptr的一个大问题是语法上开起来是复制，但是实际执行的是移动语义。试图访问一个被移动过的 auto_ptr 的成员会引发未定义的行为，所以你要特别小心，不要再去操作被移动过的 auto_ptr 。

```C++
auto_ptr<Shape> a(new Triangle);   // create triangle
auto_ptr<Shape> b(a);              // move a into b
double area = a->area();           // undefined behavior
```

但是 auto_ptr 并不总是危险的，工厂方法是一个使用 auto_ptr 的绝佳场合

```C++
auto_ptr<Shape> make_triangle()
{
    return auto_ptr<Shape>(new Triangle);
}

auto_ptr<Shape> c(make_triangle());      // move temporary into c
double area = make_triangle()->area();   // perfectly safe
```

注意两个例子是如何使用相同的句法的

```C++
auto_ptr<Shape> variable(expression);
double area = expression->area();
```

但是，他们一个出现了未定义的行为，另一个确实是安全的。那么表达式 a 和 make_triangle() 的区别在哪儿？他们难道不是相同的类型 type ？他们确实 type 相同，但是属于不同的值类型 Value categories。

### Value categories

很显然 代表 std::auto_ptr 变量 的 表达式 a 和 表示 函数调用并在每次调用都返回一个新创建的 std::auto_ptr 变量的 make_triangle() 表达式有着深层次的区别。a 是一个左值，而 make_triangle() 则是一个右值。

从左值 a 进行移动操作是危险的，因为后续我们可能会尝试调用 a 的成员函数，从而引发未定义的行为。相反，从右值 make_triangle() 则是安全的，因为在 拷贝构函数之后 我们无法在访问这个临时变量。没有代表这个临时变量的表达式。如果我们再写一遍 make_triangle() 那么会产生一个新的临时变量。实际在上在以下行代码之前，临时变量就已经销毁了。

```C++
auto_ptr<Shape> c(make_triangle());
                                  ^ the moved-from temporary dies right here
```

注意 字母 l 和 r 代表 赋值操作符的 左边 和 右边 是有历史原因的。在 C++ 里面已经不完全适用了，因为有一些不能出现在赋值操作符左边的左值（比如数组或者没有定义赋值操作符的用户自定义类型）。也有一些可以出现在左边的右值（定义了赋值操作符的右值）

>An rvalue of class type is an expression whose evaluation creates a temporary object. Under normal circumstances, no other expression inside the same scope denotes the same temporary object.

>一个类类型的右值，是一个在运算时能产生一个临时对象的表达式。在正常情况下，相同的作用域内没有其他表达式来代表这个临时对象。

### 右值引用

我们现在明白了，从左值进行移动是很危险的，而从右值引用则是很安全的。那么如果 C++ 从语言层面能支持区分 左值 参数还是 右值 参数，那么我们就能完全禁止对左值进行移动，或者至少能让对左值的移动操作变成显示的。这样可以防止意外的对左值进行移动。

C++11 提供的方案是右值引用。右值引用是只能绑定到右值上的引用，语法是 x&& 。原有的 引用 x& 现在被称为左值引用。（注意右值引用并不是引用的引用，在 C++ 里面没有这个概念）。

再加上 const 关键字，我们可以组合出四种引用类型，他们能够绑定的对象如下

```C++
            lvalue   const lvalue   rvalue   const rvalue
---------------------------------------------------------              
X&          yes
const X&    yes      yes            yes      yes
X&&                                 yes
const X&&                           yes      yes
```

实践中你可以忽略 const X&&， 限制对右值的只读访问没有多大意义。

>An rvalue reference X&& is a new kind of reference that only binds to rvalues.

### 隐式转换

右值引用也经历了好几个版本的变化，从2.1版本开始， 如果能从 Y 隐式类型转换到 X， 那么 右值引用 X&& 同样能绑定到 所有 Y 的value categories上。在这种情况下会生成一个临时对象 X ，然后右值引用绑定到这个临时对象上。

```C++
void some_function(std::string&& r);

some_function("hello world");
```

在上面的例子中， ”hello world” 是类型 const char[12] 的一个左值。既然存在从 const char[12] 的隐式类型转换（通过 const char× 到 std::string)，这儿会生成一个临时的 std::string 对象，然后 r 绑定到这个对象上。右值和临时对象的区别比较模糊，这是其中一个不同点。

### 移动构造函数

一个有用的带有 X&& 形参的方法的例子就是移动构造函数 X::X(X&& source)。 它的目的是把所管理资源的所有权从 source 转移到当前对象上。
In C++11, std::auto_ptr<T> has been replaced by std::unique_ptr<T> which takes advantage of rvalue references. I will develop and discuss a simplified version of unique_ptr. First, we encapsulate a raw pointer and overload the operators -> and *, so our class feels like a pointer:

在 C++11 里面， std::auto_ptr<T> 被使用了右值引用的 std::unique_ptr<T> 所取代。我会编写并讨论一个简单版本的 unique_ptr。首先，我们封装一个裸指针，然后实现 -> 和 * 操作符，这样我们的类才能用起来像个指针：

```C++
template<typename T>
class unique_ptr
{
    T* ptr;

public:

    T* operator->() const
    {
        return ptr;
    }

    T& operator*() const
    {
        return *ptr;
    }
```

构造函数获取指针所有权，然后析构函数释放指针指向的资源

```C++
    explicit unique_ptr(T* p = nullptr)
    {
        ptr = p;
    }

    ~unique_ptr()
    {
        delete ptr;
    }
```

下面是真正有意思的地方，移动构造函数

```C++
    unique_ptr(unique_ptr&& source)   // note the rvalue reference
    {
        ptr = source.ptr;
        source.ptr = nullptr;
    }

```

这个移动构造函数的行为和 auto_ptr 的构造函数一致，只不过他只能作用到实参是右值的情形：

```C++
unique_ptr<Shape> a(new Triangle);
unique_ptr<Shape> b(a);                 // error
unique_ptr<Shape> c(make_triangle());   // okay
```

第二行代码会编译失败，因为 a 是一个左值，而 移动构造函数的形参只能绑定到右值上。这就是我们想要的结果，永远不允许隐式执行危险的移动操作。第三行能够正常的编译，因为 make_triangle() 是一个右值。 移动构造函数会把临时对象的所有权转移到 c 里面。再说一次，这就是我们想要的结果。

>The move constructor transfers ownership of a managed resource into the current object.

### 移动赋值操作符

最后需要补充的是移动赋值操作符，它的作用是释放旧资源，然后从参数中取得新的资源的所有权

```C++
    unique_ptr& operator=(unique_ptr&& source)   // note the rvalue reference
    {
        if (this != &source)    // beware of self-assignment
        {
            delete ptr;         // release the old resource

            ptr = source.ptr;   // acquire the new resource
            source.ptr = nullptr;
        }
        return *this;
    }
};
```

注意这个函数的实现其实是重复了移动构造函数和析构函数的工作，你熟悉 copy-and-swap 惯用法么？其实对于移动语义同样适用 move-and-swap 方法

```C++
    unique_ptr& operator=(unique_ptr source)   // note the missing reference
    {
        std::swap(ptr, source.ptr);
        return *this;
    }
};
```

注意 source 是 类型 unique_ptr 的一个变量，它将会有一个移动构造函数初始化，也就是说实参会被移动到形参上。实参仍然要求必须是一个右值，因为移动构造函数本身需要一个右值的引用绑定到形参上。当代码执行到 赋值操作符的 右大括号时， source 就超出了自己的作用域，同时会自动释放旧的资源。

>The move assignment operator transfers ownership of a managed resource into the current object, releasing the old resource. The move-and-swap idiom simplifies the implementation.

### 从左值进行移动操作

有时我们希望从左值进行移动。意思是说有时候我们希望编译器把一个左值当成一个右值来用，从而可以调用移动构造函数，虽然这有时候是危险的。为了这个目的， C++11 在标准库的 <utility> 头文件中提供了一个模板方法 std::move。这个名气有一点名不副实，因为它仅仅是把一个左值转换为右值，它本身并不做任何移动操作。它仅仅让移动操作能被执行。也许他应该被命名为 std::cast_to_rvalue or std::enable_move，但是我们现在已经习惯了它现在的名字。

下面展示怎么显示的从左值进行移动

```C++
unique_ptr<Shape> a(new Triangle);
unique_ptr<Shape> b(a);              // still an error
unique_ptr<Shape> c(std::move(a));   // okay
```

注意到在第三行结束后，a 就不在拥有一个 triangle 对象了。这是没问题的，因为我们已经显示的使用了 std::move(a), 我们已经明确的表明了我们的意图：亲爱的构造函数，为了初始化 c 你可以对 a 做任何你需要的事情，我已经不在关注他了；

>std::move(some_lvalue) casts an lvalue to an rvalue, thus enabling a subsequent move.

### Xvalues

注意到 std::move(a) 虽然是一个右值，但是并不产生一个临时对象。这个难题迫使标准委员会引入了第三个 value category. 一种在传统意义上并不是右值，但是却可以绑定到右值上的类型 xvalue  (eXpiring value)。传统意义上的右值被重命名位prvalues （Pure rvalues）.

prvalues 和 xvalues 都属于 rvalues， cvalues 和 lvalues 都属于 glvalues （Generalize lvalues). 他们的关系用下图可以方便的表示

```C++
        expressions
          /     \
         /       \
        /         \
    glvalues   rvalues
      /  \       /  \
     /    \     /    \
    /      \   /      \
lvalues   xvalues   prvalues
```

只有 xvalues 是新的，其余都是原有的重命名或者分组名。

>C++98 rvalues are known as prvalues in C++11. Mentally replace all occurrences of "rvalue" in the preceding paragraphs with "prvalue".

### 移动到方法外部

直到这儿我们看到了 移动到局部变量上，移动到方法参数上。但是移动操作也可以发生在相反的方向上。如果一个方法通过值返回，在调用处会使用移动构造函数生成一个临时对象，移动构造函数的实参就是 方法 return 语句后面的表达式运算出来的临时对象（通常是一个局部变量或者临时变量，但是可以是任意类型的）。

```C++
unique_ptr<Shape> make_triangle()
{
    return unique_ptr<Shape>(new Triangle);
}          \-----------------------------/
                  |
                  | temporary is moved into c
                  |
                  v
unique_ptr<Shape> c(make_triangle());
```

有一点可能会令人感到惊讶，自动变量（没有声明为 static 的局部变量）也会自动被移动到方法外面

```C++
unique_ptr<Shape> make_square()
{
    unique_ptr<Shape> result(new Square);
    return result;   // note the missing std::move
}
```

移动构造函数为什么能接受 result 这个左值作为参数？ result 的作用域即将结束，并且他将会在栈回滚是自动释放掉。在这之后没有谁能抱怨 result 被修改了。当控制流回到调用方时，result 就不再存在了。 基于这个原因， C++11 有一个特殊的规则，允许从函数返回 自动变量，而不需要显示写 std::move。实际上你永远不应该在返回自动变量时写 std::move， 因为这回影响 NRVO 优化。

>Never use std::move to move automatic objects out of functions.

请注意在两个工厂函数中返回的都是值，而不是右值引用。右值引用也是引用，就像通常的做法，永远不要返回自动变量的引用。调用方会得到一个悬空的引用，如果你欺骗编译器去接受你的代码

```C++
unique_ptr<Shape>&& flawed_attempt()   // DO NOT DO THIS!
{
    unique_ptr<Shape> very_bad_idea(new Square);
    return std::move(very_bad_idea);   // WRONG!
}
```

>Never return automatic objects by rvalue reference. Moving is exclusively performed by the move constructor, not by std::move, and not by merely binding an rvalue to an rvalue reference.

### 移动到成员变量上

早晚，你会写下面的代码

```C++
class Foo
{
    unique_ptr<Shape> member;

public:

    Foo(unique_ptr<Shape>&& parameter)
    : member(parameter)   // error
    {}
};
```

编译器基本上都会抱怨 parameter 是一个 左值。如果看类型，你会看到一个右值引用，但是简单来说右值引用就是 一个绑定到右值上的引用。他并不是说引用的本身是个右值，本质上来说 parameter 就是一个有名字的变量。你可以在构造函数内部使用 parameter 这个变量任意次而且他每次都指向同一个对象。隐式的对他进行移动是危险的，所以语言禁止这么使用。

>A named rvalue reference is an lvalue, just like any other variable.

解决方案就是显示的激活移动语义

```C++
class Foo
{
    unique_ptr<Shape> member;

public:

    Foo(unique_ptr<Shape>&& parameter)
    : member(std::move(parameter))   // note the std::move
    {}
};
```

你也许会争辩到在member初始化完成后就没有再被访问，为什么没有一个像 返回自动变量的 特殊规则来给他自动加上 std::move?可能是因为这会加重编译器实现的负担。比如，如果构造函数体定义在另一个编译单元呢？作为对比，返回值规则仅仅需要对照符号表，确定 return 语句后面的表达式是不是代表一个自动变量就行了。

你也可以在构造函数里面采用传值的方式，目前并没有成型的惯用法来应对这种只能 move  的类型。就我个人来说，我更喜欢传值，因为这样接口会看起来更加清晰。

### 特殊成员函数

C++98 会自动在需要时生成三个特殊成员函数 析构函数，拷贝构造函数，拷贝赋值操作符。

```C++
X::X(const X&);              // copy constructor
X& X::operator=(const X&);   // copy assignment operator
X::~X();                     // destructor
```

右值引用经历了数个版本，在 3.0 版本开始，编译器会在需要时自动定义两个特殊成员函数 移动构造函数和移动赋值操作符，请注意 VC10 和 VC11 都不符合 右值引用 3.0 版本，所以你仍然需要手动添加他们

```C++
X::X(X&&);                   // move constructor
X& X::operator=(X&&);        // move assignment operator
```

这两个特殊成员函数仅在没有手动声明的情况下才会默认声明，并且如果你自定义了移动构造函数或者移动赋值操作符，拷贝构造函数和拷贝赋值操作符就不会被自动生成了。

在实践中这些规则意味着什么？

>If you write a class without unmanaged resources, there is no need to declare any of the five special member functions yourself, and you will get correct copy semantics and move semantics for free. Otherwise, you will have to implement the special member functions yourself. Of course, if your class does not benefit from move semantics, there is no need to implement the special move operations.

>如果你写的类不需要管理资源，也就没有必要声明任何五个特殊函数中的一个，而且你可以免费获取正确的拷贝和移动语义。相反，你就需要自己手动实现这些特殊的成员函数。当然如果你的类从移动语义中获取不到好处，那么也就没有必要实现特殊的移动语义函数了。

注意 拷贝赋值操作符和移动赋值操作符可以统一成一个赋值操作符。采用传值的方式传递参数

```C++
X& X::operator=(X source)    // unified assignment operator
{
    swap(source);            // see my first answer for an explanation
    return *this;
}
```

这样需要实现的特殊成员函数就成五个减少到四个了。这儿其实有异常安全和性能的 交换。但是我并不是这方面的专家。

### Forwarding references (previously known as Universal references)

考虑下面的函数模板

```C++
template<typename T>
void foo(T&&);
```

你可能仅仅希望 T&& 绑定到右值上。因为第一眼看 T&& 是一个右值引用。事实证明， T&& 依然能绑定到左值上。

```C++
foo(make_triangle());   // T is unique_ptr<Shape>, T&& is unique_ptr<Shape>&&
unique_ptr<Shape> a(new Triangle);
foo(a);                 // T is unique_ptr<Shape>&, T&& is unique_ptr<Shape>&
```

如果实参是 类型 X 的右值 ， T 会被推到为 X， 那么 T&& 意味着 X&&。 这是任何人都希望发生的事情。 但是如果实参是 类型 X 的左值，基于特殊的规则， T 被推到为 X&， 意味着 T&& 代表类似 X& &&的类型。但是既然 C++ 还没有引用的引用的概念 类型 X& && 会被降成 X&。这看起来让人迷惑而且没有什么用处，但是引用崩塌是实现完美转发的基础（这儿不在讨论这个）

>T&& is not an rvalue reference, but a forwarding reference. It also binds to lvalues, in which case  T and T&& are both lvalue references.


If you want to constrain a function template to rvalues, you can combine SFINAE with type traits:

如果你想限制函数模板值推导出右值，你可以使用类型萃取技术结合 [SFINAE](http://en.cppreference.com/w/cpp/language/sfinae)

```C++
#include <type_traits>

template<typename T>
typename std::enable_if<std::is_rvalue_reference<T&&>::value, void>::type
foo(T&&);
```

### move 的实现

现在你理解了 引用崩塌 的概念 （reference collapsing），下面就是 std::move 的实现

```C++
template<typename T>
typename std::remove_reference<T>::type&&
move(T&& t)
{
    return static_cast<typename std::remove_reference<T>::type&&>(t);
}
```

As you can see, move accepts any kind of parameter thanks to the forwarding reference T&&, and it returns an rvalue reference. The std::remove_reference<T>::type meta-function call is necessary because otherwise, for lvalues of type X, the return type would be X& &&, which would collapse into X&. Since t is always an lvalue (remember that a named rvalue reference is an lvalue), but we want to bind t to an rvalue reference, we have to explicitly cast t to the correct return type. The call of a function that returns an rvalue reference is itself an xvalue. Now you know where xvalues come from ;)

就像你看到的那样，多亏了 引用转发 T&& move 可以接受任何类型的参数，并且能返回右值引用。std::remove_reference<T>::type 元函数的调用是必须的，如果不这样， 对于类型 X 左值来说， 返回值就会是 X& && 然后仍然被降成 X&。 既然 t 永远是一个左值（记住一个命名的右值引用也是一个左值），但是我们希望把 t 绑定到一个右值引用上，我们就必须显示的把返回值转换为正确的返回类型。调用一个返回右值引用的函数的表达式本身是一个 xvalue， 现在你应该明白左值是怎么来的了。

>The call of a function that returns an rvalue reference, such as std::move, is an xvalue.

注意这儿通过右值引用的方式返回是没有问题的，因为 t 并不是代表一个自动对象，而是代表一个由调用方传递过来的对象。