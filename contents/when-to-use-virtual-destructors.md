# 何时使用虚析构函数？

## 回答

在你有可能通过基类指针去 delete 子类对象时，虚析构函数会很有用

```C++
class Base 
{
    // some virtual methods
};

class Derived : public Base
{
    ~Derived()
    {
        // Do some important cleanup
    }
};
```

我在这儿并没有把基类的析构函数声明为 virtual 的。让我们看一下如下代码片段

```C++
Base *b = new Derived();
// use b
delete b; // Here's the problem!
```

因为 基类析构函数并不是 virtual 的 同时 b 是一个 Base× 的指针 而实际指向了 Derived 对象。 这种情况下 delete b 是未定义的行为

>[In delete b], if the static type of the object to be deleted is different from its dynamic type, the static type shall be a base class of the dynamic type of the object to be deleted and the static type shall have a virtual destructor or the behavior is undefined.  
如果被 delete 的对象的静态类型和动态类型不一致。该对象的静态类型必须是动态类型的基类并且静态类型需要有虚析构函数，否则 这种情况下会发生未定义行为

在绝大多数数的实现里，这种情况下对析构函数的调用会和其他非虚代码一样，也就是说会直接调用基类的析构函数而不是子类的，从而引发资源泄露。

总的来说，如果需要使用累的多态性，那么就把基类的析构函数设置为虚函数。

如果你想禁止通过基类指针 delete 子类 对象，你可以把基类析构函数设置为 protected 和 非虚的。这样做编译器就不会允许你直接 delete 基类的指针。

你可以通过 [这篇文章](http://www.gotw.ca/publications/mill18.htm) 学习更多关于 virtual 关键字 和 虚析构函数的知识。