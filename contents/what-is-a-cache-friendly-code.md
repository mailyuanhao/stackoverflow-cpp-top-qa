# 什么是 “cache-friendly" 的代码？

"cache unfriendly code" 和 "cache friendly" 代码的区别是什么？

我怎么确保我的代码是 cache-efficent 的？

## 回答

### 预备知识

在现代计算机的 memory 体系里， 只有处于最底层的寄存器能够在单个时钟周期内实现数据移动。但是寄存器十分昂贵，大多数计算机的 cpu 核心 的寄存器数量也就小几十个（小几百到差不多一千个 byte 的容量）。而 memory 体系的另一端 （DRAM
DRAM）内存 则十分廉价（相比寄存器要便宜几百万倍），但是要从 内存获取数据则要花费数百个时钟周期。快但是贵 的 寄存器 和 便宜但是慢 的 内存之间由 cache 进行过渡。cache 分为 L1， L2， L3 三个级别速度和价格逐级递减。这样设计的理念是 大多数代码都会频繁的访问少量的变量，而其余的大量的变量只会偶尔访问。如果处理器无法在 L1 中找到数据 则会在 L2 中查找， 如果 L2 也找不到就会到 L3 中查找，最后在内存中去查找。每一次”未命中“ 查找时间成本都很昂贵。

（如果吧 cache 比作内存的话，那么内存就可以比作硬盘。硬盘存储成本特别低，但是速度也特别慢）

使用缓存 是降低延迟影响的几个主要方法之一。 用 Herb Sutter 的话来说就是：增加带宽很容易，但是延迟确无法摆脱。increasing bandwidth is easy, but we can't buy our way out of latency。

数据总是经由存储层级去获取（最小且最快的 到 最慢的）一次 cache 的命中/未命中 是指 CPU 中最高层次的 cache 的命中/未命中，这儿最高层次是指 最大且最慢的 cache。 cache 的命中率对性能来说至关重要， 因为每一次的未命中都会引发从 内存 （甚至更糟糕）获取数据，这会消耗 **大量**  的时间（从内存中需要数百个时钟周期，从 磁盘则需要消耗数千万计 的时钟周期）。 作为对比从 cache 中获取数据则只需要消耗几个时钟周期。

现在的计算机体系架构中，性能瓶颈出现在让 CPU 去等待 (leaving the CPU die) （比如访问 内存 或 更高的存储层次）。随着时间的推移，这只会越来越糟糕。处理器频率的提升现在已经和性能的提升脱钩。**问题就出在内存的的访问上**。 目前 CPU 的硬件设计工作着重于 cache 优化，预取，指令流水线以及并发上。For instance, modern CPUs spend around 85% of die on caches and up to 99% for storing/moving data!

围绕这个主题有很多值得说明的。下面是一些很棒的关于 cache， 存储层次 和 适当编码的 文章：

- [Agner Fog's page](http://www.agner.org/optimize/). 从这篇文档中你可以找到 覆盖从汇编到 C++ 语言的详细的例子。
- 如果你要视频资料，我强烈建议你看一下这个 [Herb Sutter's talk on machine architecture](http://www.youtube.com/watch?v=L7zSU9HI-6I) (youtube) (specifically check 12:00 and onwards!).
- [Slides about memory optimization by Christer Ericson](http://www.research.scea.com/research/pdfs/GDC2003_Memory_Optimization_18Mar03.pdf) (director of technology @ Sony)
- LWN.net's article ["What every programmer should know about memory"](http://lwn.net/Articles/250967/)

### cache-friendly code 的主要概念

cache-friendly 代码一个很重要的方面是关于 [局部性原则](http://en.wikipedia.org/wiki/Locality_of_reference)， 该原则的目标是把相关联的数据在内存中紧挨在一起放置来提高缓存性能。 从 CPU cache 的角度来说， 了解 cache line 并且知道他们是[如何工作](https://stackoverflow.com/questions/3928995/how-do-cache-lines-work)的是很重要的：

下面特定方面 对于 缓存优化来说特别重要：

1. 时间局部性： 当一块特定的内存地址被访问后，该地址在不久后有很大概率会再次被访问，理想的情况下再次访问时这块信息已经在缓存中了。
2. 空间局部性： 这是指把相关的数据放在临近的地址。缓存不仅仅局限于 CPU 内在很多层次上都会有。比如，当从 内存 读取数据时，并不是仅仅读取需要的数据，而是读入一块数据，因为程序很可能马上就会需要这些数据。硬盘缓存也遵循相同的方式。特别是对 CPU cache 来说， cache line 的概念特别重要。

#### 选择合适的 C++ 容器

一个简单的 cache-friendly 和 cache-unfriendly 的对比的例子是 c++ 的 std::vector 和 std::list 的对比。 vector 的元素在内存中顺序访问，而 list 的元素则放的到处都是，所以 对 vector 元素的访问要比对 list 元素的访问 更加 cache-friendly。这和空间局部性相关。

Bjarne Stroustrup 的[视频](http://www.youtube.com/watch?v=YQs6IC-vgmo) 很好的解释了这个问题。

#### 在设计数据结构和算法时不要忽略 缓存 问题

任何时候，只要有可能就要把你的数据结构以及计算顺序设计成能最大化利用 cache 的方式。一个重要的技术是 [cache blocking](http://www.cs.berkeley.edu/~richie/cs267/mg/report/node35.html)， 该技术在高性能计算领域相当重要。

#### 了解并且利用数据的内部（implicit）结构

另一个简单的例子是（相关领域的人也经常忘记）二维数组存储时的 列优先（fortran，matlab） vs 行优先 （c， c++）。比如对于下面的矩阵来说

```C++
1 2
3 4
```

在 行优先 的顺序里，这儿矩阵会按照 1 2 3 4 的顺序存储。在列优先的顺序里存储顺序则是 1 3 2 4. 可以看出如果实现不利用这种顺序，会很容易陷入（也很容易避免）缓存问题。很不幸，我发现在我的领域（机器学习）这种问题很常见。

当从内存中读取矩阵的特定元素时，周边的元素也会被读取存储到 cache line 中。如果利用这种顺序（上面提到的）这回减少内存的访问次数（因为后续计算需要的一些值已经被缓存到 cache line 中了）

简化一下，假设 cache 由一个能存储两个矩阵元素的 cache line 组成 所以当从内存中读取一个元素时，下一个也会被读入。比如说我们要计算上面 2 × 2 矩阵 所有元素的和（假设叫 M）

利用这种顺序（比如在 C++ 中优先改变 列 索引)

```C++
M[0][0] (memory) + M[0][1] (cached) + M[1][0] (memory) + M[1][1] (cached)
= 1 + 2 + 3 + 4
--> 2 cache hits, 2 memory accesses
```

不利用这种顺序，（比如在 C++ 中 优先改变 行 索引）

```C++
M[0][0] (memory) + M[1][0] (memory) + M[0][1] (memory) + M[1][1] (memory)
= 1 + 3 + 2 + 4
--> 0 cache hits, 4 memory accesses
```

在这个简单的例子中，利用这种顺序比不利用的速度要快一倍（因为内存访问的速度要远大于计算和的速度）。在实际程序中性能的差距会更明显。

#### 避免不可预测（unpredictable） 的 分支

现代架构中的流水线以及编译器在重排代码以减少内存访问方面做的越来越好。如果你的关键代码包含（unpredictable） 分支时，数据预读会边的更加困难甚至无法实现。这会引发更多的 cache 未命中问题。

[文章](https://stackoverflow.com/questions/11227809/why-is-processing-a-sorted-array-faster-than-an-unsorted-array) 很好的解释了这个问题。

#### 避免虚函数

C++ 中， 虚函数在 cache 未命中 方面有很大争议（一个普遍共识是 从性能的角度来说，应该能不用就不用）。虚函数在查找时会触发 cache 未命中问题，但是这仅仅发生在当该函数不经常被调用时（如果经常调用，那么他会被缓存在 cache 中），所以有人人认为这不是个问题。

关于这个问题可以参考[文字](https://stackoverflow.com/questions/667634/what-is-the-performance-cost-of-having-a-virtual-method-in-a-c-class)

### 常见问题

现代多 CPU cache 架构的一个常见问题被称作 [false sharing](http://en.wikipedia.org/wiki/False_sharing). 当不同的 CPU 把 不同的内存数据映射到相同的 cache line 时会触发该问题。 这会造成 cache line （存储有别的 CPU 需要的数据） 被一遍一遍的覆盖。 不同的线程会让彼此触发 cache 未命中 从而产生等待。 参看这篇[文字](https://stackoverflow.com/questions/8469427/how-and-when-to-align-to-cache-line-size)。

内存关于 缓存 失效的一个极端情况被称作 [抖动 thrashing](http://en.wikipedia.org/wiki/Thrashing_%28computer_science%29) （这很可能并不属于你说的场景）。 这发生在 处理器不停地触发页错误（比如访问并不在当前页的内存）从而引发磁盘访问时。