# 什么是 rvalues， lvalues， xvalues， glvalues 还有 prvalues ？

## 回答

>新的表达式分类是什么？

[FCD (n3092)](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2010/n3092.pdf) 已经给出了完美的说明：

>lvalue （这个叫法源于左值可以出现在赋值表达式的左边）指定一个函数或者对象。[举例： 如果 E 是一个指针类型的表达式，那么 *E 就是一个左值代表 E 指向的 对象或者函数。另一个例子，调用一个返回类型是是左值的函数 也是一个左值]  
xvalue （eXpiring value）指向一个生命周期即将结束的对象（所以它的资源可以被偷走）。一个 xvalue 是特定类型表达式的结果 包括右值引用（8.3.2） [例子：调用一个返回类型是右值引用的函数就是 xvalue]  
glvalue （generalized lvalue）包括 lvalue 和 xvalue  
rvalue（和左值类似，这么叫是因为可以出现在赋值表达式的右边）可以是一个 xvalue， 一个临时对象（12.2）或者它的子对象，或者一个没有关联到对象上的值  
prvalue （pure rvalue）不是 xvalue 的 rvalue。[比如 调用一个返回值不是引用的函数， 字面量比如 12， 7.3e5，或者 true]  
每一个表达式都属于该分类法的基本分类之一： lvalue， xvalue 或者 prvalue 。表达式的这个属性被称为它的值类型。[在 Claue 5 讨论每一种内置操作符时会指明该操作符 期望的输入输出操作数的值类型。比如内置赋值操作符期望左操作数是个左值右操作数是一个 prvalue 并且产生一个左值作为结果。用户自定义操作符属于函数，所以他们期望的输入输出操作数由他们的参数列表和返回值决定]

我建议你完整的读一下 张杰 2.10 Lvalues and rvalues 

>这些新的类别如何和已有的 rvalue 和lvalue 相关联？

![如图](https://i.stack.imgur.com/GNhBF.png)

>C++0x 中的左值和右值类型和 C++03 中的一样么？

随着 move 语义的引进 左值的语义已经进化了。

>为什么需要这些新的分类？

为了支持定义移动构造，移动赋值函数。