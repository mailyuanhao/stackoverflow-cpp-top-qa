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

