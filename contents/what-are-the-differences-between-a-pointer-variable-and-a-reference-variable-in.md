# C++ 中指针和引用有哪些区别？

## 回答 （有人对回答进行了整理）

- 指针可以被任意次的重复赋值， 但是引用一旦绑定后就不能被重新绑定。
- 指针可以指向 NULL （其实是任意地址），引用只能绑定到一个对象上。
  
  下面是译者加的
  - 故意创造指向空或者无效对象的引用也是可以的，程序员自己高兴就好。
  - 基于这一点，通过指针传入的变量都会判断是否有效（是否为*空*）,而通过引用传入的则默认是有效的。
- 引用变量本身不能被获取地址，指针变量却可以。
- 不能进行*引用算术运算*。（不过可以获取引用绑定的对象的地址后进行指针算术运算 例如： &ojb + 5）。

澄清一个容易误解的点：

    虽然 C++ 标准极力避免引导编译器对引用的实现。但是所有的编译器都是通过指针来实现引用的。也就是下面这句代码

```cpp
int &ri = i;
```

    如果没有被优化掉的话，会分配一块和指针一样大小的内存，然后把变量 i 的地址写入这块内存。

所以， 指针和引用在内存占用上是一致的。

使用的一般规则如下：

- 函数参数和返回值使用引用，这样可以提供有用且自文档化的接口。
- 实现算法和数据结构时使用指针。

推荐阅读材料：

- [C++ FAQ lite](http://yosefk.com/c++fqa/ref.html).
- [References vs. Pointers](http://www.embedded.com/electronics-blogs/programming-pointers/4023307/References-vs-Pointers).
- [An Introduction to References](http://www.embedded.com/electronics-blogs/programming-pointers/4024641/An-Introduction-to-References).
- [References and const](http://www.embedded.com/electronics-blogs/programming-pointers/4023290/References-and-const).
  
## 译者注

关于引用和指针的使用场景这只是一家之言。不同的编码规范有各自不同的理解和要求，就像代码风格没有高下之分，适合自己使用场景的就是最好的。

其实还有其他一些区别，目前想到的比如：

- 把临时对象绑定到一个引用上可以延长临时对象的生命周期， 但是直接获取临时对象的地址是很危险的 （编译器一般会警告或报错）。

[原题链接](https://stackoverflow.com/questions/57483/what-are-the-differences-between-a-pointer-variable-and-a-reference-variable-in?page=1&tab=votes#tab-top)