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


For the binary arithmetic operators, do not forget to obey the third basic rule operator overloading: If you provide +, also provide +=, if you provide -, do not omit -=, etc. Andrew Koenig is said to have been the first to observe that the compound assignment operators can be used as a base for their non-compound counterparts. That is, operator + is implemented in terms of +=, - is implemented in terms of -= etc.

According to our rules of thumb, + and its companions should be non-members, while their compound assignment counterparts (+= etc.), changing their left argument, should be a member. Here is the exemplary code for += and +, the other binary arithmetic operators should be implemented in the same way:

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
operator+= returns its result per reference, while operator+ returns a copy of its result. Of course, returning a reference is usually more efficient than returning a copy, but in the case of operator+, there is no way around the copying. When you write a + b, you expect the result to be a new value, which is why operator+ has to return a new value.3 Also note that operator+ takes its left operand by copy rather than by const reference. The reason for this is the same as the reason giving for operator= taking its argument per copy.

The bit manipulation operators ~ & | ^ << >> should be implemented in the same way as the arithmetic operators. However, (except for overloading << and >> for output and input) there are very few reasonable use cases for overloading these.

3 Again, the lesson to be taken from this is that a += b is, in general, more efficient than a + b and should be preferred if possible.

Array Subscripting
The array subscript operator is a binary operator which must be implemented as a class member. It is used for container-like types that allow access to their data elements by a key. The canonical form of providing these is this:

class X {
        value_type& operator[](index_type idx);
  const value_type& operator[](index_type idx) const;
  // ...
};
Unless you do not want users of your class to be able to change data elements returned by operator[] (in which case you can omit the non-const variant), you should always provide both variants of the operator.

If value_type is known to refer to a built-in type, the const variant of the operator should return a copy instead of a const reference.

Operators for Pointer-like Types
For defining your own iterators or smart pointers, you have to overload the unary prefix dereference operator * and the binary infix pointer member access operator ->:

class my_ptr {
        value_type& operator*();
  const value_type& operator*() const;
        value_type* operator->();
  const value_type* operator->() const;
};
Note that these, too, will almost always need both a const and a non-const version. For the -> operator, if value_type is of class (or struct or union) type, another operator->() is called recursively, until an operator->() returns a value of non-class type.

The unary address-of operator should never be overloaded.

For operator->*() see this question. It's rarely used and thus rarely ever overloaded. In fact, even iterators do not overload it.