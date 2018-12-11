# 为什么 结构体的 的 sizeof 不等同于所有成员的 sizeof 的和？

## 回答

原因是为了满足对齐要求而增加了填充空间。[Data structure alignment](http://en.wikipedia.org/wiki/Data_structure_alignment) 不仅影响程序性能而且影响程序执行的正确性。

- 不对齐的访问可能会引发硬错误（经常是 SIGBUS）。
- 不对齐的访问也可能引发软错误
  - 可能会由硬件矫正，引起适度的性能下降
  - 或者由软件仿真解决，引起严重的性能下降
  - 而且会破坏原子操作和其他并发安全的保证，从而引发一些微妙的错误

下面是一个使用典型配置的 X86 处理器的例子（32bit 或者 64bit 模式通用）：

```C++
struct X
{
    short s; /* 2 bytes */
             /* 2 padding bytes */
    int   i; /* 4 bytes */
    char  c; /* 1 byte */
             /* 3 padding bytes */
};

struct Y
{
    int   i; /* 4 bytes */
    char  c; /* 1 byte */
             /* 1 padding byte */
    short s; /* 2 bytes */
};

struct Z
{
    int   i; /* 4 bytes */
    short s; /* 2 bytes */
    char  c; /* 1 byte */
             /* 1 padding byte */
};

const int sizeX = sizeof(struct X); /* = 12 */
const int sizeY = sizeof(struct Y); /* = 8 */
const int sizeZ = sizeof(struct Z); /* = 8 */
```

可以通过把成员按照对齐要求来排序来最小化结构体的内存占用（对于基本类型来说，按照大小排序就足够了）（比如上面例子中 结构体 Z）

**重要提示** C 和 C++ 标准都规定 结构体对齐是实现定义的。所以不同的编译器可能会选择不同的对齐方式，从而生成不同且不兼容的数据布局方式。基于这个问题，对于需要适应不同编译器的库来说，了解编译器怎么处理对齐就很重要了。一些编译器可以通过命令行 和(或者) 特殊的 #pragma 语句来改变对齐设置。
