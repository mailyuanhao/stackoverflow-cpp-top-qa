# C++ 标准对 int， long 类型的长度是怎么规定的？

## 回答

C++ 标准并没有以 byte 为单位来规定内置类型的长度，取而代之标准规定了他们能够表示的最小的数值范围。你可以根据数值范围计算出这些类型的最小 bit 长度。然后使用 CHAR_BIT 宏（[定义每个 byte  的 bit 数](https://stackoverflow.com/questions/437470/type-to-use-to-represent-a-byte-in-ansi-c89-90-c/437640#437640) 计算出以 byte 为单位的长度。 CHAR_BIT 宏在大多数平台上是 8，而且不会该宏不会小于 8。

对于 char 来说还有另一个约束就是 size 肯定是 1 byte 或者说 CHAR_BIT 个 bit （根据名字也能看出来）。

[标准要求的最小范围](http://www.open-std.org/JTC1/SC22/WG14/www/docs/n1256.pdf) 如下：

[MSDN](http://msdn.microsoft.com/en-us/library/s3f49ktz.aspx) 中也有数据类型表示范围说明：

1. signed char -127 to 127 (注意 不是 -128 to 127；  this accommodates 1's-complement and sign-and-magnitude platforms)
2. unsigned char： 0 to 255
3. “plain” char： 和 signed char 或者 unsigned char 等同，[实现相关](https://stackoverflow.com/q/2397984)
4. signed short -32767 to 32767
5. unsigned short: 0 to 65535 
6. signed int: -32767 to 32767 
7. unsigned int: 0 to 65535 
8. signed long: -2147483647 to 2147483647 
9. unsigned long: 0 to 4294967295 
10. signed long long: -9223372036854775807 to 9223372036854775807 
11. unsigned long long: 0 to 18446744073709551615

C++ （或者 C）的实现可以任意设置 某种类型 type 由几个 byte 组成，只要满足下面条件即可：

1. sizeof(type) * CHAR_BIT 计算出来的 bit 数足够表示标准要求的数值范围 并且
2. 类型之间的大小相对顺序不变（比如 sizeof(int) <= sizeof(long))

具体某种实现实际 的类型的数值范围可以在如下头文件找到 C 的头文件 <limits.h> 或者 C++ 的头文件 <climits>。 或者使用 <limits> 头文件中的 模板 std::numberic_limits。

比如，下面是获取 int 类型 最大范围的方法：

C:

```C++
#include <limits.h>
const int min_int = INT_MIN;
const int max_int = INT_MAX;
```

C++:

```C++
#include <limits>
const int min_int = std::numeric_limits<int>::min();
const int max_int = std::numeric_limits<int>::max();
```
