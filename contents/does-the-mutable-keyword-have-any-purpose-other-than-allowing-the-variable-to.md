# mutable 除了允许 const function 修改变量之外，还有别的作用么？

## 回答

它可以用于区分 按位常量 (bitwise const ) 还是 逻辑常量 (logical const)。 逻辑常量指对象不会被通过公开接口而出现可见的改变，比如lock。另一个例子是一个类在第一次时被调用时计算变量结果，然后缓存这个结果供以后使用。

从 C++11 开始 mutable 可以在 lambda 表达式上使用来指示通过 value 捕获的可以被修改（默认是不允许的）

```C++
int x = 0;
auto f1 = [=]() mutable {x = 42;};  // OK
auto f2 = [=]()         {x = 42;};  // Error: a by-value capture cannot be modified in a non-mutable lambda
```
