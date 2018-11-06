# C++ 操作符重载有哪些规则和原语（模式做法？通行做法？ idioms）

## 回答

### C++ 操作符重载的一般语法

C++ 操作符重载只能作用于用户自定义类型（1）， 不能改变内置类型操作符的含义。意思是说要重载的操作符的操作数中至少有一个是用户自定义类型。类似其他的重载函数，操作符对于一组特定的参数类型只能重载一次。

并不是所有的 C++ 操作符都能被重载，不能重载的操作符如下： . :: sizeof typeid .* 和三元操作符 ?: 。

C++ 能够被重载的操作符如下：

- 算术操作符 + - * / % 和 += -= *= /= %= (二元中缀操作符); + - (一元前缀); ++ -- (一元前缀，后缀)。
- 比特位处理操作符: & | ^ << >> and &= |= ^= <<= >>= (二元中缀); ~ (一元前缀)。
- 布尔运算: == != < > <= >= || && (二元中缀); ! (一元前缀)
- 内存管理: new new[] delete delete[]
- 隐私转换操作符
- 其他操作符: = [] -> ->* , (二元中缀); * & (一元前缀) () (函数调用, n-ary infix)

虽然你可以重载上述所有的操作符，并不意味着你应该这样做。操作符重载的一般规则如下：

C++ 操作符重载形式类似于 有特殊名称的函数。操作符重载通常实现为左操作数的成员函数或者非成员函数。是可以选择哪一种还是只能使用特定的某种形式依赖于一些条件（2）。应用于对象 x 的一元操作符 @ （@本身不是有效操作符，代称），可以使用 x.operator@() 形式调用也可以使用 operator！(x) 形式调用。二元中缀操作符 @ 作用于 对象 x， y可以使用如下两种方式调用 operator@(x, y), x.operator @(y)。

非成员函数形式的操作符重载，通常是操作数类的友元函数。

>1 用户自定义类型这个术语有可能会有歧义。 C++ 区分 内建类型和用户自定义类型。 前者包含 int， char， double等类型。后者包含所有的j结构体， 类， 联合和枚举类型，包括标准库定义的类型，虽然他们看起来并不是”严格的用户自定义“.

>2 后续有回答进行讲解.

### C++ 操作符重载的三个基本原则

C++ 操作符重载有三条你要遵守的基本原则。虽然每种规则都确实存在例外的情况，有时违反这些规则编写的代码也算坏代码，但是这些例外都是少数情况而且各有不同。在我遇到的情况来说至少99%的背离规则的情况是不合理的，甚至说999‰.所以说，你最好准守这些规则。

1. 除非操作符的作用清晰明确毫无争议，否则不要重载。使用一个良好命名的函数替代重载。
  >操作符重载的最基本最核心最首要的原则就是不要重载他们。这听起来很奇怪，因为有很多关于操作符重载的知识，也有很多文章，书籍来阐释他们。但是实际上仅有极其少数的情况才适用操作符重载。除非某个操作符在程序所在的领域是广为人知而且没有争议的，否则很难理解程序中操作符背后的实际意图。与通常的认知相反，这种适用情况极少出现。

2. 永远不要改变操作符的通常语义。
  >C++ 对于操作符重载的语义没有任何限制。编译器欣然接受把二元 + 操作符实现为对右操作数的减法操作。但是操作符的用户却不可能想到 x + y 是从 y 中减去 x。当然这假定在程序所应用的领域，操作符的作用是无异议的。

3. 相关联的一组操作符要同时提供。
  >操作符彼此之间是相关联的，如果提供了 a + b， 用户就希望能使用 a += b， 如果支持前缀自增 ++a， 用户同样希望能使用后置自增 a--。如果可以进行 a < b 的检查， 用户同样希望能进行 a > b的比较。如果实现了拷贝构造，那也应该实现赋值操作符。

### 操作符重载实现为成员函数还是非成员函数的选择依据

二元操作符 = （赋值）， [] （数组下标）， -> （成员访问）以及 () （函数调用） 操作符语法要求必须实现为成员函数。

其余的操作符可以实现为成员也可以实现为非成员。但是有一些通常要被实现为非成员函数，因为他们的左操作数类代码你无法修改。这一点表现最明显的是输入输出操作符，他们的左操作数是 标准库提供的 stream 类。

下面几条经验法则可以帮助你做选择：

1. 一元操作符，实现为成员函数。
2. 二元操作符如果对两个操作数做相同的操作（并不修改他们）， 实现为非成员函数。（If a binary operator treats both operands equally (it leaves them unchanged), implement this operator as a non-member function.）
3. 二元操作如果对两个操作数施加不同的操作（通常会修改左操作数），那么为了方便修改左操作数的私有成员，把它实现为成员函数更方便。（If a binary operator does not treat both of its operands equally (usually it will change its left operand), it might be useful to make it a member function of its left operand’s type, if it has to access the operand's private parts.）

当然所有的经验法则都有例外情况，比如

```C++
enum Month {Jan, Feb, ..., Nov, Dec}
```

如果你想给 Month 实现 自增 自减 操作符，你只能实现为非成员函数，因为语法不允许 枚举 有成员函数。 再比如 给 模板类的内嵌模板类实现 operator< 时， 在类体内直接实现会更易于代码的编写和阅读。但是这些列外都极少出现。

(如果你想让操作符的最左面的操作数是 const 的， 那么非成员函数第一个参数要被声明为 const 的， 而成员函数版本要直接把函数声明为 const 来把 *this 改变为 const 引用)

### 常见运算符重载

操作符重载的大部分工作都是写模式化的代码，这一点也不奇怪，因为操作符实际的工作都可以被（而且基本上也是转发给）普通函数实现。但是你必须保证这些模式化的代码正确，否则要么你的代码编译不过，要么用户的代码编译不过，要么会造成用户代码出现奇怪行为。

#### 赋值操作符

关于赋值操作符有很多需要说明的，不过[GMan's famous Copy-And-Swap FAQ](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom)中涵盖了绝大多数的内容，所以这儿就略过不说了。仅仅给出操作符重载的样例代码作参考：

```C++
X& X::operator=(X rhs)
{
  swap(rhs);
  return *this;
}
```

#### 位移操作符（用于 Stream 输入输出的）

位移操作符 << 和 >>, 虽然小范围内还在用于处理比特位，但是在绝大多数程序中常见用法是重载为 stream 的输出，输入操作符。下面的 二元算术操作符 章节会详细说明位移相关的操作，本章节主要说明如何使用 iostreams 来格式化和解析你的对象。

像其他常见重载的操作符那样，stream 操作符 属于二元操作符，语法并不限制是实现为成员函数还是非成员函数。他们需要修改左操作数（改变流的状态）根据经验法则应该实现为左操作数的成员函数。但他们的左操作数是标准库提供的 streams，所以你给自己的类型实现输出输入时只能定义为非成员函数。（标准库内部大多数的输入输出操作确实实现为 流 的成员函数了）。样例实现代码如下

```C++
std::ostream& operator<<(std::ostream& os, const T& obj)
{
  // write obj to stream

  return os;
}

std::istream& operator>>(std::istream& is, T& obj)
{
  // read obj from stream

  if( /* no valid object of T found in stream */ )
    is.setstate(std::ios::failbit);

  return is;
}
```

输入操作符实现过程中唯一需要手动修改 stream 状态的情况是，数据读取操作成功了，但是结果不符yuan合预期。

#### 函数调用操作符

函数调用操作符用于创建函数对象（也被称为仿函数），只能是成员函数。所以他有一个隐含的 this 参数。除了这个参数，他可以被重载为包含任意个其他参数（可以是零个）。

下面展示一个示例：

```C++
class foo {
public:
    // Overloaded call operator
    int operator()(const std::string& y) {
        // ...
    }
};
```

调用方式:

```C++
foo f;
int a = f("hello");
```

在标准库中，仿函数在使用过程中都是直接 复制 的。所以你自己的仿函数必须保证 复制 操作足够轻量。如果一个仿函数的确需要使用复制开销很大的数据时，不好把这些数据放到对象本身内部。

#### 比较操作符

依赖经验法则，二元比较操作符实现为非成员函数，一元前缀 ! 操作符实现为成员函数。（并不建议重载这个操作符）。

标准库中的算法（比如 std::sort())和类型(比如 std::map)只需要对象实现 operator< 就足够了。但是基于一般规则如果实现了 operator< 其他的比较操作符也要一并实现。

```C++
inline bool operator==(const X& lhs, const X& rhs){ /* do actual comparison */ }
inline bool operator!=(const X& lhs, const X& rhs){return !operator==(lhs,rhs);}
inline bool operator< (const X& lhs, const X& rhs){ /* do actual comparison */ }
inline bool operator> (const X& lhs, const X& rhs){return  operator< (rhs,lhs);}
inline bool operator<=(const X& lhs, const X& rhs){return !operator> (lhs,rhs);}
inline bool operator>=(const X& lhs, const X& rhs){return !operator< (lhs,rhs);}
```

需要注意的是仅仅只需要实现两个操作符，其他的操作可以转发给这两个操作符来实现。

二元布尔操作符（||， &&)的重载同样遵循比较操作符，但是你通常没有足够的理由去重载他们。内置版本的 || && 支持 shortcut 语义，但是自定义的版本因为实质上是函数调用语法糖所以是不支持 shortcut 语义的。但是客户代码可能需要，甚至依赖这个特性，所以最好永远不要重载这两个操作符。

#### 算术操作符

##### 一元算术操作符

一元自增，自减操作符有前置后置两个版本，为了作区分，后置版本需要一个额外 int 型的参数。如果重载自增自减操作符，务必前后置的都要提供。

```C++
class X {
  X& operator++()
  {
    // do actual increment
    return *this;
  }
  X operator++(int)
  {
    X tmp(*this);
    operator++();
    return tmp;
  }
};
```

注意后置版本使用前置版本的实现，而且后置版本多了一次复制操作。虽然对内置类型来说编译器会对后置版本做优化，从而性能上和前置版本一致。但是对于自定义类型，编译器无法做优化。所以除非必要推荐使用前置版本，因为习惯性使用后置版本后会习惯性的对自定义类型也是用后置版本，这样效率较低。

重载一元前置加，减操作符很少见，也不推荐实现。如果需要，请重载为成员函数。

##### 二元算术操作符

二元算术操作符同样要遵循第三条经验法则，如果提供 +， 同时提供 +=,如果提供了 -, 同样要提供 -= 等等。 据说 Andrew Koenig 是第一个注意到可以基于复合赋值操作符来实现对应的普通版本。比如 可以基于 += 来实现 +， 基于 -= 来实现 -。 

基于我们的经验规则， + 和其他 （companions） 二元操作符要实现为非成员函数， 而赋值复合版本，因为要修改左操作数，所以应该实现为成员函数。下面是 + 和 += 的实现示例，别的应该和下面的示例类似实现。

```C++
class X {
  X& operator+=(const X& rhs)
  {
    // actual addition of rhs to *this
    return *this;
  }
};
inline X operator+(X lhs, const X& rhs)
{
  lhs += rhs;
  return lhs;
}
```

+= 操作符返回引用，而 + 返回结果的拷贝。返回引用肯定效率更高，但是 + 操作 需要返回一个新的值，所以只能返回拷贝（所以为了效率应该优先考虑 += 而不是 + ）。注意 + 操作符的左操作数是传值而不是传引用，这和 operator = 的是一个原因。

位操作操作运算符  ~ & | ^ << >> 应该参考算术运算符的实现方法。（除非 重载为 输出，输入操作符的 << 和 >>），这几个运算符极少需要重载。

#### 数组下标操作符

数组下标操作符是必须被实现为成员的二元操作符。它主要用于可以通过键值（key）来访问数据成员的 容器 类型。标准实现方法如下

```C++
class X {
        value_type& operator[](index_type idx);
  const value_type& operator[](index_type idx) const;
  // ...
};
```

两个版本的 operator[] 应该同时提供，如果你不想让用户修改 operator[] 的返回值，可以删掉 non-const 版本。

如果 value_type 确定是内置类型的引用， const 版本应该改为返回值的拷贝而非引用。

#### 类指针类型使用的运算符(Operators for Pointer-like Types)

如果要编写 迭代器 或者 智能指针， 你需要重载 解引用操作符 * 和 成员访问操作符 ->

```C++
class my_ptr {
        value_type& operator*();
  const value_type& operator*() const;
        value_type* operator->();
  const value_type* operator->() const;
};
```

这两个操作符大多情况下需要同时提供 const 和 non-const 版本。

-> 有一个特殊的 "drill-down behavior" 行为

```C++
struct client
    { int a; };

struct proxy {
    client *target;
    client *operator->() const
        { return target; }
};

struct proxy2 {
    proxy *target;
    proxy &operator->() const
        { return * target; }
};

void f() {
    client x = { 3 };
    proxy y = { & x };
    proxy2 z = { & y };

    std::cout << x.a << y->a << z->a; // print "333"
}
```

一元取地址操作符务必不要重载。

关于 -> 和->* 可以参考[这个问题](https://stackoverflow.com/questions/8777845/overloading-member-access-operators-c)

#### 类型转换操作符 （也被称为用户自定义转换）

C++ 允许你创建允许编译器进行类型转换的操作符。这种类型转换操作符分为 隐式 （implicit） 和 显示 （explicit） 两种。

##### 隐式类型转换操作符 （C++98/C++03 and C++11）

隐式类型转换允许编译器自动（implicitly）把用户自定义类型转为另一种类型（类似 int 转换为 long）。

下面示例一个实现隐式转换的简单的类

```C++
class my_string {
public:
  operator const char*() const {return data_;} // This is the conversion operator
private:
  const char* data_;
};
```
隐私类型转换类似单参数构造函数属于用户自定义转换。编译器在尝试匹配重载函数时会自动采用该转换。

```C++
void f(const char*);

my_string str;
f(str); // same as f( str.operator const char*() )
```

这个看起来很有用，但是隐式转换会在预期不到的时候默默执行。下面的示例代码 中 void f(const char*) 将会被调用，因为 my_string() 不是一个左值，所以第一个 f 函数无法匹配 

```C++
void f(my_string&);
void f(const char*);

f(my_string());
```

C++ 新手甚至于有经验的 C++ 程序员有时也会困惑于编译器选择了非预期的重载函数。显示类型转换可以缓解这个问题。

##### 显示类型转换（C++11）

与隐式类型转换不同，显示转换只有在你明确要求是才会进行。下面是一个简单的例子

```C++
class my_string {
public:
  explicit operator const char*() const {return data_;}
private:
  const char* data_;
};
```
采用显示版本后，再尝试运行上述的测试代码，会报编译错误

>prog.cpp: In function ‘int main()’:  
prog.cpp:15:18: error: no matching function for call to ‘f(my_string)’  
prog.cpp:15:18: note: candidates are:  
prog.cpp:11:10: note: void f(my_string&)
prog.cpp:11:10: note:   no known conversion for   argument 1 from ‘my_string’ to ‘my_string&’  
prog.cpp:12:10: note: void f(const char*)  
prog.cpp:12:10: note:   no known conversion for argument 1 from ‘my_string’ to ‘const char*’

要执行显示类型转换，你必须使用 static_cast， C 风格转换符，或者 constructor style cast ( i.e. T(value) )。

下面两段是关于显示转换的另一个优势，感觉只翻译这两段话很难说明白，直接把原文附在下面，可以详细参考 [Safe Bool idiom](https://www.artima.com/cppsource/safebool.html)来理解下面这两段话的意思。

>However, there is one exception to this: The compiler is allowed to implicitly convert to bool. In addition, the compiler is not allowed to do another implicit conversion after it converts to bool (a compiler is allowed to do 2 implicit conversions at a time, but only 1 user-defined conversion at max).

>Because the compiler will not cast "past" bool, explicit conversion operators now remove the need for the Safe Bool idiom. For example, smart pointers before C++11 used the Safe Bool idiom to prevent conversions to integral types. In C++11, the smart pointers use an explicit operator instead because the compiler is not allowed to implicitly convert to an integral type after it explicitly converted a type to bool.

### 重载 new 和 delete

>注意本文只关注重载 new 和 delete 的语法而非实现。相关的内容应该在另一个 [FAQ](https://stackoverflow.com/questions/7149461/) 中。

#### 基本用法

C++中， 当类似 new T(arg) 的表达式被执行时，实际上要做两件事情。首先调用 operator new 来分配原始内存空间，然后调用适当的 T 的构造函数来把这块内存构造为有效的对象。类似的，当调用 delete 删除一个对象时，先调用析构函数，然后调用 operator delete 释放内存。

C++ 允许你调整 内存管理 和 对象的 构造，析构 这两个操作。调整构造，析构通过给类编写构造析构函数实现。优化内存管理可以通过自定义 operator new 和 operator delete 来实现。

操作符重载的第一原则（不要重载）尤其适用于 new 和 delete。重载这两个操作符唯一的理由是性能问题和内存受限问题或者修改内存管理算法能带来更大的收益。

```C++
void* operator new(std::size_t) throw(std::bad_alloc); 
void  operator delete(void*) throw(); 
void* operator new[](std::size_t) throw(std::bad_alloc); 
void  operator delete[](void*) throw(); 
```

前两个用于给单个对象分配释放内存，后两个供对象数组使用。如果你提供了自己的版本，他们不会重载而是替换标准库中的版本。

如果你重载了 operator new，同时一定要提供对应的 operator delete， 即使你不打算调用它。因为如果在使用 new 表达式时 构造函数抛出了异常，运行库会自动调用对应的 operator delete 来释放已经分配的内存，如果你不提供的话，默认的版本会被调用，

如果重载了 new 和 delete， 对应的数组版本也要重载。

#### Placement new

placement new，delete 是带有额外参数的 new， delete。通过调用他们可以实现在你指定的内存地址上构造对象。

```C++
class X { /* ... */ };
char buffer[ sizeof(X) ];
void f()
{
  X* p = new(buffer) X(/*...*/);
  // ...
  p->~X(); // call destructor
} 
```

C++标准库同样附带重载版本的 Placement new 和 delete

```C++
void* operator new(std::size_t,void* p) throw(std::bad_alloc); 
void  operator delete(void* p,void*) throw(); 
void* operator new[](std::size_t,void* p) throw(std::bad_alloc); 
void  operator delete[](void* p,void*) throw(); 
```

注意：上面给出的 placement new 例子里面， operator delete 仅在 X 的构造函数抛异常时才会被调用。

你也可以使用别的参数重载 new 和 delete。像 placement new 的附加参数一样，这些参数也被列在 new 关键字后面的括号里面。因为历史原因这种 版本 也被称为 placement new。虽然他们的参数并不是为了在特定的地址构造对象。

#### 特定类的 new 和 delete

进行内存管理优化的一般原因是 测试结果 显示某一特定 class 或者某一组 classes 的对象进行频繁的创建的销毁，而为了通用目的优化的运行库中的内存管理算法在这种特定情况下效率较低。为了改善效率，你可以给特定的 类 重载 new 和 delete。

```C++
class my_class {
  public:
    // ...
    void* operator new();
    void  operator delete(void*,std::size_t);
    void* operator new[](size_t);
    void  operator delete[](void*,std::size_t);
    // ...
};
```

这些重载的 new 和 delete 类似于静态成员函数。对 my_class 的对象来说， std::size_t 参数始终是 sizeof(my_class)。当然这些操作符也会被用来动态创建继承的类的对象，此时 std::size_t 参数就可能比 sizeof(my_class)要大。

#### 全局 new 和 delete

为了重载全局 new 和 delete， 可以直接替换标准库提供的预定义的操作符。但是，这种情况极少遇到。

## 注意

关于 new 和 delete 重载 More Effective C++ 中有细致的讲解。

[原题链接](https://stackoverflow.com/questions/4421706/what-are-the-basic-rules-and-idioms-for-operator-overloading/4421791#4421791)