# C++11 总的 lambda 表达式 是什么？什么时候使用？他们能解决什么问题？

## 回答

### lambda 之前的问题

C++ 包含一些很便利的泛型函数比如 std::for_each 和 std::transform.不幸的是他们使用起来可能会很繁琐，尤其是当你使用的 [仿函数](https://stackoverflow.com/questions/356950/what-are-c-functors-and-their-uses)各不相同时。

```C++
#include <algorithm>
#include <vector>

namespace {
  struct f {
    void operator()(int) {
      // do something
    }
  };
}

void func(std::vector<int>& v) {
  f f;
  std::for_each(v.begin(), v.end(), f);
}
```

如果你仅仅在这个特定的地方使用 f 一次。那么编写一个完整的类就仅仅为了完成一次特定细微的工作，就显得有些多余了。

在 C++03 中 你可能想写下面的代码，来保证 f 是局部的

```C++
void func2(std::vector<int>& v) {
  struct {
    void operator()(int) {
       // do something
    }
  } f;
  std::for_each(v.begin(), v.end(), f);
}
```

但是在 C++03 中这是行不通的，f 不允许传递给[模板方法](https://en.cppreference.com/w/cpp/language/function_template)。

### 新的解决方案

C++11 引入了 lambda 从而允许你编写 内嵌，匿名的仿函数来替代 struct f。对于短小简单的代码片段来说，它可以让代码更加整洁易读也可以提高可维护性，比如下面的例子中

```C++
void func3(std::vector<int>& v) {
  std::for_each(v.begin(), v.end(), [](int) { /* do something here*/ });
}
```

lambda 仅仅是 匿名函数的语法糖。

### 什么是 lambda 函数？

C++ 中的 lambda 函数 源于 lambda 运算和 函数式编程。当你有一小段无法重用也没有必要做成普通函数时，lambda 函数就特别有用（不仅仅是理论上的，事件中也是如此）

C++ 中的 lambda 函数的定义像下面这样

```C++
[](){ } //barebone lambda
```

或者完成形式

```C++
[]() mutable -> T { } // T is the return type, still lacking throw()
```

[]是捕获列表， () 是参数列表， {} 是函数体

#### 捕获列表

捕获列表定义了 哪些外部值可以在 lambda 内以何种方式被访问。 可能是

- 一个变量值： [x]
- 一个变量的引用： [&x]
- 以引用的方式访问当前作用域内的所有变量：[&]
- 和上一条一样，只不过以值的方式：[=]

你可以使用 逗号 分割 任意组合。

#### 参数列表

等同于 C++ 普通函数的参数列表

#### 函数体

当 lambda 被调用时 实际执行的代码

#### 返回类型推导

如果 lambda 函数仅有一个 return 语句， 返回类型可以省略，会采用默认返回类型 decltype(return_statement)。

#### mutalbe

允许 lambda 函数修改以 值 类型捕获的数据。

#### 实际使用例子

标准库从 lambda 表达式受益颇多，并且可用性得到很大的提升，因为用户不用再把一些代码片段组装成一个个可访问的小函数了。

### C++14

C++14 中 lambda 根据若干提案进行了扩展。

#### 带初始化器的捕获列表

捕获列表中的某一项可以通过 = 进行初始化。这样可以对变量重命名或者实现通过 move 的方式捕获。标准中的一个例子

```C++
int x = 4;
auto y = [&r = x, x = x+1]()->int {
            r += 2;
            return x+2;
         }();  // Updates ::x to 6, and initializes y to 7.
```

下面是从 维基 中引用的用于展示 move 捕获的例子

```C++
auto ptr = std::make_unique<int>(10); // See below for std::make_unique
auto lambda = [ptr = std::move(ptr)] {return *ptr;};
```

#### 泛型 lambda

lambda 也可以泛型（auto 相当同于 T， 如果 T 在外层作用域中是一个模板类型参数）

```C++
auto lambda = [](auto x, auto y) {return x + y;};
```

#### 改进的返回值推导

C++14 允许所有的函数都能进行返回值推导，并且不受限于 形如 return expression;的函数。这一点也适用于 lambda。

请参考 [Lambda 表达式](https://zh.cppreference.com/w/cpp/language/lambda)
