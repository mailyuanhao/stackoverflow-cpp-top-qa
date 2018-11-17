# 为什么 模板 的实现只能写在头文件中？

引用 [The C++ standard library: a tutorial and handbook](http://books.google.com/books?id=n9VEG2Gp5pkC&pg=PA10&lpg=PA10&dq=%22The%20only%20portable%20way%20of%20using%20templates%20at%20the%20moment%20is%20to%20implement%20them%20in%20header%20files%20by%20using%20inline%20functions.%22&source=bl&ots=Ref8pl8dPX&sig=t4K5gvxtBblpcujNxodpwMfei8I&hl=en&ei=qkR6TvbiGojE0AHq4IzqAg&sa=X&oi=book_result&ct=result&resnum=3&ved=0CC8Q6AEwAg#v=onepage&q=%22The%20only%20portable%20way%20of%20using%20templates%20at%20the%20moment%20is%20to%20implement%20them%20in%20header%20files%20by%20using%20inline%20functions.%22&f=false) 的原话：

>The only portable way of using templates at the moment is to implement them in header files by using inline functions.

为什么？

（澄清， 使用头文件并不是唯一可行的方案，但确实是最简单遍历的方案）

portable solution ？？ 怎么翻译？

## 回答

把模板 实现 放在头文件中并不是必须的，答案的末尾会给出另一个方案。

在编译器实例化模板时，会使用提供的模板参数创建一个新类：

```C++
template<typename T>
struct Foo
{
    T bar;
    void doSomething(T param) {/* do stuff using T */}
};

// somewhere in a .cpp
Foo<int> f; 
```

到这一行代码时，编译器会生成一个和示例等同的新类（比如叫FooInt）

```C++
struct FooInt
{
    int bar;
    void doSomething(int param) {/* do stuff using int */}
}
```

编译器为了使用这个模板参数进行实例化（这儿是 int），就必须要知道到模板方法的实现。如果这些实现不在头文件中，编译器就无法访问到他们，也就无法进行实例化操作了。

一个通行的解决方案是在头文件中声明模板，然后把实现统一定义到一个实现文件中（比如.hpp），然后在头文件的最后 inlcude 这个实现文件。

```C++
// Foo.h
template <typename T>
struct Foo
{
    void doSomething(T param);
};

#include "Foo.tpp"

// Foo.tpp
template <typename T>
void Foo<T>::doSomething(T param)
{
    //implementation
}
```

采用这个方法，实现和声明可以进行分离，同时满足了编译器访问的需要。

另一个方案是，分离实现后同时显示的实例化你所有可能需要的实例。确切的说在一个 能访问模板实现的源文件中实例化他们。

```C++
// Foo.h

// no implementation
template <typename T> struct Foo { ... };

//----------------------------------------    
// Foo.cpp

// implementation of Foo's methods

// explicit instantiations
template class Foo<int>;
template class Foo<float>;
// You will only be able to use Foo with int or float
```

如果我的回答不够明了，你可以参考这篇文章 [C++ Super-FAQ on this subject](https://isocpp.org/wiki/faq/templates#templates-defn-vs-decl) 。

