# 什么是对象切片？

## 回答

“切片” 是指你把子类的对象赋值给基类的对象，从而造成造成部分信息被 “切” 掉。

比如

```C++
class A {
   int foo;
};

class B : public A {
   int bar;
};
```

一个 B 类的对象会有两个成员 foo 和 bar。

然后如果你写下面的代码

```C++
B b;
A a = b;
```

会造成 b 中的 bar 成员的信息在 a 中丢失。
