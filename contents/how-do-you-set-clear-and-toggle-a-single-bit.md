# 怎样对单个比特位进行设置，清除， 取反 操作？

## 回答

### 设置比特位

使用 比特运算符 OR (|)

```C++
number |= 1UL << n;
```

把 number 的第 n + 1 个比特位设置为1。

如果 number 比 unsigned long 要宽， 则使用 1ULL 。 因为 1UL << n 在求值后才会进行提升， 而对一个 long 做超过位宽的移位操作是未定义的。后面的例子同样使用这个规则。

### 清除比特位

使用 比特运算符 AND (&)

```C++
number &= ~(1UL << n);
```

### 比特位取反

使用 比特运算符 XOR (^)

```C++
number ^= 1UL << n;
```

### 检查一个比特位

```C++
bit = (number >> n) & 1U;
```

### 把第 n 个比特位设置为 x

基于 C++ 的二补码实现，下面代码可以实现把第 n 个比特位设置为 x

```C++
number ^= (-x ^ number) & (1UL << n);
```

如果 x 不是 1 或者 0， 结果是无效的。

如果不依赖二补码的行为，可以使用负无符号数

```C++
number ^= (-(unsigned long)x ^ number) & (1UL << n);
```

或者

```C++
unsigned long newbit = !!x;    // Also booleanize to force 0 or 1
number ^= (-newbit ^ number) & (1UL << n);
```

通常来说在位操作中使用无符号数是个好主意。

使用封装好的预处理器宏比直接拷贝粘贴代码要好，比如：

```C++
/* a=target variable, b=bit number to act upon 0-n */
#define BIT_SET(a,b) ((a) |= (1ULL<<(b)))
#define BIT_CLEAR(a,b) ((a) &= ~(1ULL<<(b)))
#define BIT_FLIP(a,b) ((a) ^= (1ULL<<(b)))
#define BIT_CHECK(a,b) (!!((a) & (1ULL<<(b))))        // '!!' to make sure this returns 0 or 1

/* x=target variable, y=mask */
#define BITMASK_SET(x,y) ((x) |= (y))
#define BITMASK_CLEAR(x,y) ((x) &= (~(y)))
#define BITMASK_FLIP(x,y) ((x) ^= (y))
#define BITMASK_CHECK_ALL(x,y) (((x) & (y)) == (y))   // warning: evaluates y twice
#define BITMASK_CHECK_ANY(x,y) ((x) & (y))
```

### 关于一补码，二补码

x 的一补码 就是 [反码](https://zh.wikipedia.org/zh-cn/%E4%B8%80%E8%A3%9C%E6%95%B8)。

x 的二补码就是我们常说的[补码](https://zh.wikipedia.org/wiki/%E4%BA%8C%E8%A3%9C%E6%95%B8)。

