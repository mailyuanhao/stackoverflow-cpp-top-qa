# const int*, const int* const 和 int const* 的区别是什么？

## 回答

["Clockwise/Spiral Rule"](http://c-faq.com/decl/spiral.anderson.html) 可以用来分析大多数的 C， C++ 声明。

- int* 指向 int 的指针
- int const * 指向 const int 的指针
- int const * const 指向 const int 的 const 指针

第一个 const 关键字可以出现在类型的左侧或者右侧所以：

- const int * == int const *
- const int * const == int const * const

你甚至可以做更疯狂的事情

- int ** 指向 指向 int 指针 的 指针
- int ** const 指向 指向 int 指针的 const 指针
- int * const * 指向 指向 int const 型 指针的指针
- int const ** 指向 指向 const int 的指针的指针
- int * const * const 指向 指向 int 的 const 指针 的 const 指针
- ...

确保我们清楚 const 的含义

```C++
const int* foo;
int *const bar; //note, you actually need to set the pointer 
                //here because you can't change it later ;)
```

foo 是一个变量指针指向一个常量的整形。它允许你变更指向的变量（修改指针本身的内容）而不允许修改指向的值。这种情况经常出现在 C-style 字符串上，你声明一个指向 const char 的指针。你可以变更你指向的字符串，但是不能修改指向字符串的内容。当字符串存储在程序的数据区段时这就很重要了，因为它是不允许修改的。

bar 是一个指向 整形的 常量指针，指向不允许变更。这就像是没有语法糖的引用。基于这个事实，在你使用 T* const 时应该优先选择使用引用，除非你型需要 NULL 指针。

### Clockwise/Spiral Rule

从变量的名字开始，顺时针螺旋查找下一个指针或者类型，重复这个步骤到表达式结束。

下面是 例子

![例子](https://i.stack.imgur.com/sT6ng.png)
![例子](https://i.stack.imgur.com/Zt0G2.png)
![例子](https://i.stack.imgur.com/kXH8P.png)
![例子](https://i.stack.imgur.com/UeqZO.png)
![例子](https://i.stack.imgur.com/f5ftV.png)