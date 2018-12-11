# C++ 调用基类构造函数需要遵守什么规则？

## 回答

如果基类构造函数没有参数，它们会被自动调用。如果你想调用有参数的基类构造函数，你必须使用子类构造函数的初始化列表。不像 Java, C++ 支持多继承（无论好坏）所以基类必须使用名字指明，而不能使用类似 "super()" 的写法。

```C++
class SuperClass
{
    public:

        SuperClass(int foo)
        {
            // do something with foo
        }
};

class SubClass : public SuperClass
{
    public:

        SubClass(int foo, int bar)
        : SuperClass(foo)    // Call the superclass constructor in the subclass' initialization list.
        {
            // do something with bar
        }
};
```

[这儿](http://www.cprogramming.com/tutorial/initialization-lists-c++.html) 和 [这儿](http://www.parashift.com/c++-faq-lite/ctors.html#faq-10.6) 有关于构造函数初始化列表的描述。