# C++ rule of three 原则是什么意思？

- 对象拷贝是什么意思？
- 拷贝构造和拷贝赋值是什么意思？
- 我应该怎么定义他们？
- 怎么禁止对象拷贝？

## 回答

> rule of three 是指 如果某个类定义了 拷贝构造，赋值操作，析构函数 中的一个，另外两个也要一并定义。

### 前言

C++ 中用户自定义类型的对象是 **值语义**。这意味这些对象在各种上下文是默认 拷贝 行为的。有必要说明 对象拷贝 的具体含义：

从一个简单的类开始

```C++
class person
{
    std::string name;
    int age;

public:

    person(const std::string& name, int age) : name(name), age(age)
    {
    }
};

int main()
{
    person a("Bjarne Stroustrup", 60);
    person b(a);   // What happens here?
    b = a;         // And here?
}
```

### 特殊成员函数

person 对象的 copy 意味着什么？ 在上面的 main 函数里面 展现了两种不同的语义。 person b（a） 调用 person 的构造函数，基于已有的对象 a 构造一个全新的对象 b。 b = a 由 赋值函数完成，因为目标对象 b 已经是一个有效的对象了，需要对 b 做一些 额外的操作才行。

person 中没有用户自定义的 拷贝构造函数，赋值操作符和析构函数，所以编译器会自动生成默认的实现。标准引文如下：
>The [...] copy constructor and copy assignment operator, [...] and destructor are special member functions. [ Note: The implementation will implicitly declare these member functions for some class types when the program does not explicitly declare them. The implementation will implicitly define them if they are used. [...] end note ] [n3126.pdf section 12 §1]  
>拷贝构造函数，拷贝赋值操作符和析构函数是一类特殊的成员函数，如果程序没有类显示的声明，实现（编译器）会自动提供默认的声明。当他们被调用时，会生成默认的实现。

默认情况下，拷贝对象就是拷贝他们的数据成员：
>The implicitly-defined copy constructor for a non-union class X performs a memberwise copy of its subobjects. [n3126.pdf section 12.8 §16]  
>The implicitly-defined copy assignment operator for a non-union class X performs memberwise copy assignment of its subobjects. [n3126.pdf section 12.8 §30]  
>对于非 union 的类 X, 默认的拷贝构造和拷贝赋值操作是对子对象做基于成员的拷贝。

### 默认实现

给 person 类 特殊成员函数提供的默认实现 类似下面的代码：

```C++
// 1. copy constructor
person(const person& that) : name(that.name), age(that.age)
{
}

// 2. copy assignment operator
person& operator=(const person& that)
{
    name = that.name;
    age = that.age;
    return *this;
}

// 3. destructor
~person()
{
}
```

在上述情况下基于成员的复制，会把成员 name 和 age 都进行复从而生成一个新的独立的，自包含的 person 对象，此时是符合我们的预期的。析构函数的默认实现永远是 空 。因为上述例子中构造函数并没有获取任何额外的资源所以默认的析构函数 空 实现也是符合预期的。在 person 的析构函数执行后会自动调用成员的析构函数。

>After executing the body of the destructor and destroying any automatic objects allocated within the body, a destructor for class X calls the destructors for X's direct [...] members [n3126.pdf 12.4 §6]

### 资源管理

当我们的类需要管理资源时（类的对象 负责 管理 资源),就需要显示的定义这些特殊的成员函数了。此种情况经常意味着 构造函数 获取了某种资源（或者某种资源传入到构造函数中）而 析构函数 负责释放。

让我们回到 标准C++(pre-standard C++) 之前的时代， 那时并没有 std::string 程序员偏爱使用指针。 那时的 person 类可能看起来是下面的样子：

```C++
class person
{
    char* name;
    int age;

public:

    // the constructor acquires a resource:
    // in this case, dynamic memory obtained via new[]
    person(const char* the_name, int the_age)
    {
        name = new char[strlen(the_name) + 1];
        strcpy(name, the_name);
        age = the_age;
    }

    // the destructor must release this resource via delete[]
    ~person()
    {
        delete[] name;
    }
};

```

即使在今天，仍然有人写这种风格的代码从而陷入问题当中，“我把 person 放入到 vector 当中，然后出现了令人抓狂的内存问题”（vector 内部 管理 对象时 是用的 对象 拷贝）。 记住！ 默认的情况下拷贝对象就是拷贝成员，而 拷贝 name 成员仅仅是拷贝了指针而不是它指向的字符数组！这会产生如下几种负面影响：

1. a 对象改变该字符数组，会影响到 对象 b.
2. 一旦 b 对象析构了， a 中的 name 指针就是 危险的 悬空指针.
3. 如果 a 发生了析构，此时会去 delete 悬空指针，这种行为是未定义的.
4. 因为在复制之前并没有考虑到 name 指向的字符数组的复制，所以这段代码早晚会出现各种恼人的内存问题.

### 显示定义

既然基于成员的复制并不能达到我们的预期，那么只能显示的定义拷贝构造函数和赋值拷贝操作符，来对对象执行深拷贝，避免内存问题:

```C++
// 1. copy constructor
person(const person& that)
{
    name = new char[strlen(that.name) + 1];
    strcpy(name, that.name);
    age = that.age;
}

// 2. copy assignment operator
person& operator=(const person& that)
{
    if (this != &that)
    {
        delete[] name;
        // This is a dangerous point in the flow of execution!
        // We have temporarily invalidated the class invariants,
        // and the next statement might throw an exception,
        // leaving the object in an invalid state :(
        name = new char[strlen(that.name) + 1];
        strcpy(name, that.name);
        age = that.age;
    }
    return *this;
}
```

注意初始化和复制的区别，复制之前必须先销毁 name 指向的旧的资源，以防止内存泄漏发生。同时要考虑 类似 x=x 这样的自我复制情况。如果不做这种检查，自我复制时，delete[] 会删除 源字符串，因为这种情况下 this.name 和 that.name 均指向同一块内存。

### 异常安全

很不幸，在内存资源紧张时，new char 会抛出异常，上述解决方案就不会出问题。一个可选的解决方案是引入一个临时变量，对上述复制代码语句进行重新排序：

```C++
// 2. copy assignment operator
person& operator=(const person& that)
{
    char* local_name = new char[strlen(that.name) + 1];
    // If the above statement throws,
    // the object is still in the same state as before.
    // None of the following statements will throw an exception :)
    strcpy(local_name, that.name);
    delete[] name;
    name = local_name;
    age = that.age;
    return *this;
}
```

上述代码同样可以避免显示的自我复制的判断。另一种更鲁邦的做法是使用 [copy-and-swap idiom](https://stackoverflow.com/questions/3279543/)。这儿并不过多的涉及 异常安全 问题。这里提到异常只是想带出下面观点： 编写管理资源的类不是一件简单的事情。

### 无法复制的资源

有些资源是无法复制的比如 文件句柄 mutex 等。此时可以把拷贝构造函数和复制操作符声明为私有的并且不提供实现，来防止类对象被复制：

```C++
private:

    person(const person& that);
    person& operator=(const person& that);
```

你也可以直接 继承 boost::noncopyable 或者使用 C++ 11 新引入的语法

```C++
person(const person& that) = delete;
person& operator=(const person& that) = delete;
```

### The rule of three

有时你需要编写类来管理资源（永远不要在一个类中管理多个资源，那只会带来无尽的痛苦）,此时 记住 The rule of three

>If you need to explicitly declare either the destructor, copy constructor or copy assignment operator yourself, you probably need to explicitly declare all three of them.

(很不幸，这个原则并没有被 C++ 标准 或者以我所知的任一编译器强制要求)

### 建议

大多数情况下，你并不需要自己在类中手动管理资源，因为已经有一些类（类似 std::string)去帮你做了。 对比上面采用 std::string 的代码和采用 指针的 晦涩易出错的代码后，你会坚信这个结论的。只要你远离 裸指针资源，你在写代码时就不用过多的考虑The rule of three。
（译者，很多裸资源比如 Windows 上的句柄， C 标准库的 文件句柄，不过 C++ 标准库等库中对这些资源都有封装可以直接使用，即使 Windows 句柄这种也可以利用 智能指针，做生命期的管理）