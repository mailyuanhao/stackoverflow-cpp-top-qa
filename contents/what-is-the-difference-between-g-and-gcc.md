# g++ 和 gcc 有什么区别？

## 回答1

gcc 和 g++ 都是 GNU 编译器集合（曾经仅仅是 GNU C  编译器）的 compiler-driver。

即使他们在不手动制定 -x language 参数的情况下能自动根据文件类型决定调用的后端程序（ cc1 cc1plus..），他们也存在一些区别。

他们之间最大的区别可能就是默认自动链接的库。

根据 GCC 的 在线文档 [link options](https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html) 和 [how g++ is invoked](https://gcc.gnu.org/onlinedocs/gcc/Invoking-G_002b_002b.html)。 g++ 等同于 gcc -xc++ -lstdc++ -shared-libgcc (第一个参数是编译器参数，后面的两个是连接器参数)。可以通过 -v 参数来验证（该参数可以显示后台运行的工具链命令行）。

## 回答2

GCC ： GNU 编译器集合

> GNU 编译器 所支持的所有语言的编译器集合

gcc: GNU C 编译器
g++: GNU C++ 编译器

主要的区别：
1. gcc 会把 *.c/*.cpp 文件分别作为 C 和 C++ 编译。
2. g++ 会同时把 *.c/*.cpp 当做 C++ 来编译。
3. 如果你使用 g++ 来链接 目标文件，他会自动链接 C++ 标准库（gcc 就不会）。
4. gcc 编译 C 文件 会使用更少的预定义 宏。
5. gcc 编译 *.cpp 和 g++ 编译 *.c/*.cpp 会定义额外的宏。

编译 *.cpp 时额外的宏

```C++
#define __GXX_WEAK__ 1
#define __cplusplus 1
#define __DEPRECATED 1
#define __GNUG__ 4
#define __EXCEPTIONS 1
#define __private_extern__ extern
```
