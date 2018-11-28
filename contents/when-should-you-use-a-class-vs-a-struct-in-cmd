# C++ 中什么情况下选择使用 struct 或者 class ？

## 回答

class 和 struct 在 C++ 中的区别是：struct 拥有默认 public 的成员和基类而 class 拥有默认 private 的成员和基类。class 和 struct 都可以同时拥有 public， protected 和 private 的成员，可以使用继承，可以拥有成员的方法。

我建议使用 struct 作为 没有任何 class-like 特性的 POD 类型。使用 class 作为拥有 private 数据和成员函数的聚合数据结构。
