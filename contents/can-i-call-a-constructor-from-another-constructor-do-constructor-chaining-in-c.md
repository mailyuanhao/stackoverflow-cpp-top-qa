# C++ 可以在一个构造函数内部调用另一个构造函数么？

## 回答

### C++11 YES！

C++11 之后有了 [代理构造函数](https://secure.wikimedia.org/wikipedia/en/wiki/C++11#Object_construction_improvement) 特性。

语法和 C# 有点小区别

```C++
class Foo {
public: 
  Foo(char x, int y) {}
  Foo(int y) : Foo('a', y) {}
};
```

### C++03 No！

很不幸 C++03 没有任何方法能实现这个功能，不过有两种方案来模拟这个功能

1. 你可以使用默认参数来合并两个（或更多）的构造函数  
```C++
class Foo {
public:
  Foo(char x, int y=0);  // combines two constructors (char) and (char, int)
  // ...
};
```
2. 使用 init 函数来共享代码  
```C++
class Foo {
public:
  Foo(char x);
  Foo(char x, int y);
  // ...
private:
  void init(char x, int y);
};

Foo::Foo(char x)
{
  init(x, int(x) + 7);
  // ...
}

Foo::Foo(char x, int y)
{
  init(x, y);
  // ...
}

void Foo::init(char x, int y)
{
  // ...
}
```

请参考[the C++ FAQ entry](https://isocpp.org/wiki/faq/ctors#init-methods)
