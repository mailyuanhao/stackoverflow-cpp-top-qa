# 为什么 switch 语句内部不能声明变量？

我一直想知道，为什么不能在 switch 语句的 case label后面声明变量？在 C++ 中你几乎可以在任何地方声明变量（当然在第一次使用的附近声明是个好习惯），为什么下面的代码编译会报错？

```C++
switch (val)  
{  
case VAL:  
  // This won't work
  int newVal = 42;  
  break;
case ANOTHER_VAL:  
  ...
  break;
}  
```

MSVC 会产生如下错误：

>initialization of 'newVal' is skipped by 'case' label

看起来其他语言也有这个限制，为什么这会是个问题？

## 回答

case 语句仅仅是个 label，这意味着编译器仅仅把它编译成一个能够跳转到的 label。在 C++ 中，这个问题属于 作用域的范畴。你的 大括号 把整个 switch 语句内部定义成一个作用域。也就是说这个作用域内部有能够直接跳转到的 label，这种跳转可能会跨过变量的初始化过程。正确的做法是在 case 内部声明一个作用域，然后把你的变量声明在这个作用域内。

```C++
switch (val)
{   
case VAL:  
{
  // This will work
  int newVal = 42;  
  break;
}
case ANOTHER_VAL:  
...
break;
}
```
