# C++ 中 **explicit** 关键字是什么意思？

## 答案

当进行函数参数解析时，编译器允许进行一次隐式类型转换。意思是编译器可以调用 **单参数** 构造函数把实参转换成另一种类型以匹配形参的类型。

下面的例子中 Foo 类的构造函数可以进行隐式类型转换

```cpp
class Foo
{
public:
  // single parameter constructor, can be used as an implicit conversion
  Foo (int foo) : m_foo (foo) 
  {
  }

  int GetFoo () { return m_foo; }

private:
  int m_foo;
};
```

下面的函数需要一个 Foo 类型的参数：

```cpp
void DoBar (Foo foo)
{
  int i = foo.GetFoo ();
}
```

下面例子展示 DoBar 的调用：

```cpp
int main ()
{
  DoBar (42);
}
```

上面代码中实参是 int 类型， 但是类 Foo 中的单参数构造函数接受一个 int 类型，可以用来进行类型转换，把 int 类型转换为 Foo 类型 （生成类 Foo 的一个临时对象）。

对于函数的每一个参数编译器都可以进行一次这种转换。

在单参数构造函数前面加上 explicit 可以禁止这个构造函数用于隐式类型转换。 加上 DoBar（42） 将会调用失败，只能增加显示的类型转换才可以 DoBar (Foo (42))。

使用这个关键字主要是为了避免出现一些隐含的 bug。

### 注意点

- 不仅仅是单参数构造函数，那些有默认值的多参数构造函数也可以被用于隐式类型转换， 比如 Object( const char* name=NULL, int otype=0)。
- 建议给所有这种构造函数默认添加 explicit 关键字。除非设计上需要隐式类型转换才去除该关键字。 
- C++11 后 explicit 关键字也可以用于修饰类型转换操作符。

### 译者注

- 很多规范书籍中都推荐使用 explicit 关键字， 比如 Google C++ 编码规范， Effective C++ 等。
- 关于类型转换 C++ Primer 中有详细的讲解。