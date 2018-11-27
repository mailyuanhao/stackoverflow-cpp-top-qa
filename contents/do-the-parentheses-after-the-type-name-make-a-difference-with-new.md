# 类型名后面加 括号 会引发不一样的 new 么？

如果 Test 是一个普通的类，下面两个代码片段会有什么不同么？

```C++
Test* test = new Test;
```

和

```C++
Test* test = new Test();
```

## 回答

让我们变得学究起来，因为它确实会影响代码的行为。

new 返回的内存有时候会被初始化有时候不会，取决于 new 的类型是不是 [POD](https://stackoverflow.com/questions/146452/what-are-pod-types-in-c) 类型或者是一个包含 POD 类型的类但是使用编译器默认提供的构造函数。

C++ 1998 存在两种初始化 清零 或者 默认值

C++ 2003 增加了第三种初始化 值初始化

假设如下情况

```C++
struct A { int m; }; // POD
struct B { ~B(); int m; }; // non-POD, compiler generated default ctor
struct C { C() : m() {}; ~C(); int m; }; // non-POD, default-initialising m
```

使用 C++98 的编译器，会发生如下情况：

new A - 不定的值

new A() - 使用 零 初始化

new B - 默认构造函数 (B::m 未定义)

new B() - 默认构造函数 (B::m 未定义)

new C - 默认构造函数 (C::m 零 初始化)

new C() - 默认构造函数 (C::m 零 初始化)

使用 C++2003 兼容的编译器，会发生如下情况：

new A - 不定值
new A() - 使用 值初始化 A， 因为 A 是 POD 类型，所以是零初始化

new B - 默认初始化 (B::m 处于未初始化状态)

new B() - 值初始化 B ，使用 零 初始化 B 的所有成员，因为没有用户自定义构造函数，所以编译器默认生成。

new C - 默认初始化 C， 调用 默认构造函数

new C() -  默认初始化 C， 调用 默认构造函数

所以在所有的 C++ 版本里面， 如果 A 是一个 POD 类型， new A 和 new A() 都是有区别的。

new B() 的情况在 C++98 和C++03 也是不同的。

这是 C++ 能把你逼疯的最阴暗的角落之一。在构建一个对象时，有时你期望，需要括号，有时不需要，有时无所谓。

## 回答2

new A() 明确指明，你需要调用构造函数来初始化内存， new A 暗示你并不介意不调用构造函数。
