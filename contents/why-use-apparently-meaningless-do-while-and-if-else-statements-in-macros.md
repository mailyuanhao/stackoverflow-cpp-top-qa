# 为什么宏定义中要使用看起来毫无意义的 do-while 或者 if-else 语句？

我看到在很多 C/C++ 代码的宏定义使用了无意义的 do while 循环进行包裹，比如下面的例子

```C++
#define FOO(X) do { f(X); g(X); } while (0)
#define FOO(X) if (1) { f(X); g(X); } else
```

我看不出来 do while 有什么作用？为什么要使用它们，而不像下面这样直接写呢？

```C++
#define FOO(X) f(X); g(X)
```

## 回答

do ... while 和 if ... else 的作用是让你的宏后面的 分号 始终有相同的意义。比如说我们定义一个类似你第二个例子的宏

```C++
#define BAR(X) f(x); g(x)
```

假如你要在一个 if 语句没有使用 大括号包裹的 if ... else 中使用上面的宏，结果可能会让你很惊讶

```C++
if (corge)
  BAR(corge);
else
  gralt();
```

上面的代码会展开如下

```C++
if (corge)
  f(corge); g(corge);
else
  gralt();
```

这是错误的语法，因为 else 已经无法和 if 对应了。使用 大括号包裹你的宏定义也是解决不了问题的，因为大括号后面跟分号在语法上是错误的。

```C++
if (corge)
  {f(corge); g(corge);};
else
  gralt();
```

这个问题有两个解决方法。第一个是在宏中使用逗号进行分割，而不会让它具有表达式的意义（he first is to use a comma to sequence statements within the macro without robbing it of its ability to act like an expression）

```C++
#define BAR(X) f(X), g(X)
```

使用上面版本的 BAR 时，代码会被扩展成如下语法正确的代码

```C++
if (corge)
  f(corge), g(corge);
else
  gralt();
```

如果你的宏里面的 *f(X)* 替换为更复杂的需要独立成块的代码片段，比如需要声明局部变量，更通用的方法是使用 类似 *do...while*  的方式把整个宏包装成独立语句，这样后面的分号就不会引发混乱了。

```C++
#define BAR(X) do { \
  int i = f(X); \
  if (i > 4) g(i); \
} while (0)
```

使用 do ... while 并不是唯一的选择，你也可以使用 if ... else。不过当在 if ... else 里面展开时，会引发 [dangling else](http://en.wikipedia.org/wiki/Dangling_else), 这会造成已经发生的 dangling else 问题更难被发现，比如下面的代码：

```C++
if (corge)
  if (1) { f(corge); g(corge); } else;
else
  gralt();
```

问题点是在不能使用悬空分号的地方使用了分号（The point is to use up the semicolon in contexts where a dangling semicolon is erroneous. ）。当然可能（也应该）争辩说，在这种情况下， BAR 应该声明为一个方法更合适。

总的来说， do...while 是用来绕开 C 预处理器的缺陷的。这也是 C 风格编码规范要求不要使用 C 预处理器时担心的一个问题点。