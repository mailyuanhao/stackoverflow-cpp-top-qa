# C/C++ 中 --> 是什么操作符？

## 问题

读完 comp.lang.c++.moderated 上的这篇[Hidden Features and Dark Corners of C++/STL](https://groups.google.com/forum/#!msg/comp.lang.c++.moderated/VRhp2vEaheU/IN1YDXhz8TMJ)文章后， 下面这段能使用 Visual Studio 2008 和 G++ 4.4进行编译并正常工作的代码让我感到很惊讶。

代码如下：

```cpp
#include <stdio.h>
int main()
{
    int x = 10;
    while (x --> 0) // x goes to 0
    {
        printf("%d ", x);
    }
}
```

这段代码同样能通过GCC的编译，所以我猜测它是在C语言中定义。请问它是由哪个标准在哪儿进行定义的？

## 回答

    请注意
    --> 并不是一个操作符， 它实际上是两个操作符 -- 和 > 分别在 C++03 标准的 §5.2.6/2 和 §5.9 中进行定义。

循环条件中的代码每次递减 x 同时返回 x 的原值，然后使用 > 操作符对 x 的原值和 0 进行比较。

## 其他答主提供的花式玩法

```cpp
while (x --\
            \
             \
              \
               > 0)
     printf("%d ", x);
```

```cpp
int x = 10;

while( 0 <---- x )
{
   printf("%d ", x);
}
```

```cpp
int x = 100;

while( 0 <-------------------- x )
{
   printf("%d ", x);
}
```

[原题链接](https://stackoverflow.com/questions/1642028/what-is-the-operator-in-c)