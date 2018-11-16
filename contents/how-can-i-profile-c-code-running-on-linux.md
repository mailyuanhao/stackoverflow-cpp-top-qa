# 如何对运行在 Linux 平台上的 C++ 代码进行 性能分析？

## 回答

### 调用栈采样法

如果你想使用性能分析工具，那么从建议的工具中选择一个就好了

不过如果你比较着急而且可以在程序运行中看起来比较慢的时候用调试器中断程序的运行，这儿有个简单的方法来查明性能问题点。

在程序运行中中断几次，每次看一下调用栈。如果某段代码浪费了 20% 或者 50% 或者 其他任意概率的时间，那么这个概率也是你在每次中断调用栈中看到它的概率。也差不多是在所有中断调用栈中能看到它的概率。并不需要事先猜测问题点。当然如果你已经猜测了可能出现性能问题的点，这个方法也可以证明你猜测的是对是错。

你的代码可能有多处严重程度不同的性能问题，解决任何一个，都会增加余下问题的浪费时间的百分比，这会让它们在后续的过程中更容易被定位。在性能问题错综复杂时，这种放大作用可以指向真正严重的性能问题点。

注意：程序员在亲身实践这个方法之前，大多倾向于对它持怀疑态度。他们会说性能检测软件可以提供同样的信息，但是这种说法仅仅在这些工具检查了所有可能的调用栈，然后允许你随机查看样本的前提下才成立（hey will say that profilers give you this information, but that is only true if they sample the entire call stack, and then let you examine a random set of samples.）。 (The summaries are where the insight is lost.)  调用图同样不能给你提供这些信息：

1. 他们无法从指令级进行分析 另外
2. 在有递归时，结果可能具有迷惑性。

他们还会说，这个方法只适用于 玩具代码，并不能适用于真正的项目中。其实在项目中使用就会发现，项目越大，该方法越好用。因为真正的项目会存在更多的性能问题需要解决。他们还会说，这个方法会发现不是问题的问题点，但是这仅仅在你只看这段代码一次时是对的。如果这段代码被发现了多次，那么他就是个真正的问题。

P.S. 如果有方法在一个时间点搜集所有的调用栈样本的话，这个方法同样适用于多线程程序。比如 JAVA。

P.P.S. 大体上说，你软件的抽象层次越多，你越有可能找到性能问题点，从而优化程序性能。

### profiling 工具推荐

#### Valgrind

你可以用如下参数调用 [Valgrind](http://en.wikipedia.org/wiki/Valgrind)。

```shell
valgrind --tool=callgrind ./(Your binary)
```

这会生成 callgrind.out.x 文件，用 kcachegrind 工具分析这个文件，可以生成图形化的结果，标示出每行代码的消耗。

#### gprof

如果你用的是 GCC ,可以使用 [gprof](http://www.math.utah.edu/docs/info/gprof_toc.html)工具

编译时加上 -pg 参数

```shell
cc -o myprog myprog.c utils.c -g -pg
```

#### perf_events

新的内核中会自带了新的性能工具 比[perf-tool](https://en.wikipedia.org/wiki/Perf_(Linux))