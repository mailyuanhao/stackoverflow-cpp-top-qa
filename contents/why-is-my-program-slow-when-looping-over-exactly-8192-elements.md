# 为什么我的程序在循环遍历 8192 个元素时会变慢？

下面是从出问题的程序中抽取的代码片段。 矩阵 img[] 的大小是 SIZE * SIZE, 并且按如下方式进行初始化：

```C++
img[j][i] = 2 * j + i
```

然后，新建一个矩阵 res[][], 并设置该矩阵的每个成员为它对应的 img 矩阵中上下左右 9 个位置的成员的 平均值。为了简化代码，位于边上的所有成员全部设置为 0

```C++
for(i=1;i<SIZE-1;i++) 
    for(j=1;j<SIZE-1;j++) {
        res[j][i]=0;
        for(k=-1;k<2;k++) 
            for(l=-1;l<2;l++) 
                res[j][i] += img[j+l][i+k];
        res[j][i] /= 9;
}
```

这就是该程序全部的内容。为了完整起见，下面给出前面的代码，后面没有别代码了。如你所见，前面仅仅是初始化代码

```C++
#define SIZE 8192
float img[SIZE][SIZE]; // input image
float res[SIZE][SIZE]; //result of mean filter
int i,j,k,l;
for(i=0;i<SIZE;i++) 
    for(j=0;j<SIZE;j++) 
        img[j][i] = (2*j+i)%8196;
```

基本上说，当 SIZE 是 2048 的整数倍时，程序运行的会比较慢，比如下面结果

```
SIZE = 8191: 3.44 secs
SIZE = 8192: 7.20 secs
SIZE = 8193: 3.18 secs
```

使用的 编译器 是 GCC。据我所知应该是和内存管理有关，但是这方面我不是很擅长，所以提出了这个问题。

能解释为什么运行时间会出现这种现象么？，如果能给出解决方案就更好了。

我知道 malloc 、 free ，但是现在的问题并不是使用的内存大小，而是运行时间的问题。我不知道还能提供什么线索。

## 回答

和下面相关问题一样，运行时间差异都是由 super-aligment 引起的。

- [Why is transposing a matrix of 512x512 much slower than transposing a matrix of 513x513?](https://stackoverflow.com/q/11413855/922184)
- [Matrix multiplication: Small difference in matrix size, large difference in timings](https://stackoverflow.com/q/7905760/922184)

你这段代码会触发这个现象，完全是下面的原因：

从最初的循环开始

```C++
for(i=1;i<SIZE-1;i++) 
    for(j=1;j<SIZE-1;j++) {
        res[j][i]=0;
        for(k=-1;k<2;k++) 
            for(l=-1;l<2;l++) 
                res[j][i] += img[j+l][i+k];
        res[j][i] /= 9;
}
```

首先内部两个循环是没有必要的，可以展开如下

```C++
for(i=1;i<SIZE-1;i++) {
    for(j=1;j<SIZE-1;j++) {
        res[j][i]=0;
        res[j][i] += img[j-1][i-1];
        res[j][i] += img[j  ][i-1];
        res[j][i] += img[j+1][i-1];
        res[j][i] += img[j-1][i  ];
        res[j][i] += img[j  ][i  ];
        res[j][i] += img[j+1][i  ];
        res[j][i] += img[j-1][i+1];
        res[j][i] += img[j  ][i+1];
        res[j][i] += img[j+1][i+1];
        res[j][i] /= 9;
    }
}
```

这样就只剩下我们感兴趣的两个外层循环了。

现在可以看出该题目中的问题和下面这个题目是一模一样的 [Why does the order of the loops affect performance when iterating over a 2D array?](https://stackoverflow.com/q/9936132/922184)

你遍历矩阵使用的是列优先而不是行优先

---

如下交换外层循环的顺序可以解决这个问题

```C++
for(j=1;j<SIZE-1;j++) {
    for(i=1;i<SIZE-1;i++) {
        res[j][i]=0;
        res[j][i] += img[j-1][i-1];
        res[j][i] += img[j  ][i-1];
        res[j][i] += img[j+1][i-1];
        res[j][i] += img[j-1][i  ];
        res[j][i] += img[j  ][i  ];
        res[j][i] += img[j+1][i  ];
        res[j][i] += img[j-1][i+1];
        res[j][i] += img[j  ][i+1];
        res[j][i] += img[j+1][i+1];
        res[j][i] /= 9;
    }
}
```

这样就完全去除了非顺序访问，你就不会遇到 SIZE 是 2 的大的幂时性能降低的问题

---

Core i7 @ 3.5 GHz

原始代码

```
8191: 1.499 seconds
8192: 2.122 seconds
8193: 1.582 seconds
```

交换循环顺序的代码

```
8191: 0.376 seconds
8192: 0.357 seconds
8193: 0.351 seconds
```

### 注释

这个问题是因为 cpu cache 组相关 策略造成的。可以参考[文章](https://cenalulu.github.io/linux/all-about-cpu-cache/) 和 [文章](https://blog.csdn.net/dongyanxia1000/article/details/53392315)