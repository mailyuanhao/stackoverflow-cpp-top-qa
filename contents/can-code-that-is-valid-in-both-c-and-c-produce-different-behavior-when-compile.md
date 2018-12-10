# C 和 C++ 同时有效的一段代码，在分别使用两种语言的编译器编译后会产生不同的行为么？

C 和 C++ 是有一些区别的，并不是所有 C 有效的代码在 C++ 中同样有效。（这儿说的有效，我是指有确定行为的标准代码，而不是实现相关或者未定义等类似的行为）。

是否存在一段在 C 和 C++ 都有效的代码，在使用两者的编译器编译后能产生不同的行为？

为了让这个对比 合理、有用 （我想学一些实用的东西，而不是找漏洞）做以下假设：

- 不要和预处理器相关联（也就是说不要使用类似 #ifdef __cplusplus, pragmas 等做区分）
- 任何 由具体实现规定的 在两种语言里面要一致（比如 数值类型的长度限制等）
- 只比较有意义的近期 版本的标准（比如 C++98 和 C90 或者更新的）。如果和版本相关，请指明哪个具体版本的标准产生了不同的行为。

## 回答1

下面 C， C++ 都有效的代码，在两个语言下 i 的值 很可能不一致。

```C++
int i = sizeof('a");
```

具体参考 [Size of character ('a') in C/C++](https://stackoverflow.com/questions/2172943/size-of-character-a-in-c-c).

这篇 [文章](http://david.tribble.com/text/cdiffs.htm) 给出的另一个例子

```C++
#include <stdio.h>

int  sz = 80;

int main(void)
{
    struct sz { char c; };

    int val = sizeof(sz);      // sizeof(int) in C,
                               // sizeof(struct sz) in C++
    printf("%d\n", val);
    return 0;
}
```

## 回答2

下面这个例子利用了两种语言 函数调用 和 对象声明机制的不同 以及 C90 允许调用未声明函数的特点。

```C++
#include <stdio.h>

struct f { int x; };

int main() {
    f();
}

int f() {
    return printf("hello");
}
```

C++ 中不会输出任何东西，因为 是进行 创建然后丢掉了 f 的临时对象的操作， 而 C90 则会输出 hello， 因为可以调用未声明的函数。

你可能会怀疑为什么 命名 f 可以被使用两次。 C 和 C++ 的标准都明确允许这样做，如果你想创建对象，你可以 明确的写 struct f 来表明你想要结构体，或者不写 struct 如果你想进行函数调用。

## 回答3

C++ vs C90 的话，至少有一种方法来制造 和实现无关的 不同行为的方法。C90 没有单行注释。 精心构造的话，我们可以利用这一点来制造一个表达式在 C90 和 C++ 中行为完全不一致。

```C++
int a = 10 //* comment */ 2
        + 3;
```

在 C++ 里面，所有 // 之后的都属于注释，所以上面表达式和下面一致

```C++
int a 10 + 3；
```

因为 C90 没有单行注释， 只有 /* comment */ 属于注释。 第一个 /  和 2 都属于表达式的一部分， 所以和下面类似

```C++
int a = 10 / 2 + 3;
```

所以一个正确的 C++ 编译器会得出 13， 但是一个严格的 C90 编译器结果会是 8. 当然我只是随机挑选了这几个数字，你可以使用任何你觉得合适的数字。

## 回答4

C90 vs C++11 (int vs double)

```C++
#include <stdio.h>

int main()
{
  auto j = 1.5;
  printf("%d", (int)sizeof(j));
  return 0;
}
```

在 C 语言里面 auto 意味着局部变量。 C90 里面忽略变量和函数类型是允许的，默认都是 int。 在 C++ 11 里面 auto 的意思完全不同，它表示让编译器根据变量被初始的值来推断变量的类型。

## 回答5

下面的例子突出了预处理器的不同

```C++
#include <stdio.h>
int main()
{
#if true
    printf("true!\n");
#else
    printf("false!\n");
#endif
    return 0;
}
```

C 会打印 “false” 而 C++ 会打印 “true”。 C 里面所有未定义宏都会被评估为 0， 而 在 C++ 里面有一个例外， “true” 会被评估为 1.

## 评分低于 100 的不再进行翻译。
