# C++11 内存模型

C++11 引入了标准内存模型，它到底是什么意思？它会怎么影响 C++ 程序编码设计？

## 回答

首先，你要学会从 语言律师(Language Lawyer) 的角度去考虑问题。

C++ 标准并不涉及任何具体的编译器，操作系统或者 CPU。他只讨论一个基于真实系统抽象出来的 “虚拟机器”。 从语言律师的角度看，程序员的工作是给这个 “虚拟机器” 编写代码，而具体的编译器则负责把这些代码在真实系统上实现。严格遵照 C++ 标准编码，你可以确定不论是现在还是50年后，你的代码可以正常的在任何系统上编译运行只要使用和该系统兼容的编译器。

C++98、C++03的标准中讨论的 ”虚拟机器“ 是单线程的，你不可能仅仅依据标准就可以编写可移植的多线程代码。该标准没有涉及任何关于内存 存取 的原子性或者相对顺序的内容，更不要说 mutex 这类的概念了。

当然，你可以给特定的系统编写多线程程序比如 pthreads 或者 Windows。但是没有标准的方法来给 C++98、C++03 的 ”虚拟机器“ 编写多线程代码。

C++11 标准定义的 ”虚拟机器“ 是多线程的并且包含一个良好定义的内存模型，也就是说它规定了编译器在进行内存访问时什么可以做，什么不可以做。

考虑下面的例子，两个全局变量被两个线程 并发 访问

```C++
           Global
           int x, y;

Thread 1            Thread 2
x = 17;             cout << y << " ";
y = 37;             cout << x << endl;
```

线程 2 将会输出什么样的结果？

对 C++98，C++03 来说，这个问题不仅仅是未定义的，而且是无意义的。因为标准中并没有 任何 线程 相关的概念。

对 C++11 来说结果是未定义的。因为 x，y的访问并不是原子性的。这看起来并没有任何改善。

不过 C++11 中你可以写如下的代码：

```C++
           Global
           atomic<int> x, y;

Thread 1                 Thread 2
x.store(17);             cout << y.load() << " ";
y.store(37);             cout << x.load() << endl;
```

越来越有意思了，现在结果是确定得了，它可以是0， 0， 也可以是37， 17，也可以是 0， 17。

它不可能的输出是 37， 0。因为 C++11 中默认的 atomic 的 load 和 store 执行顺序是强制和执行顺序一致的。也就是说 load 和 store 的执行顺序和你在各个线程中编写代码的顺序是一致的，当然线程之间这些操作是任意交错的。所以对 atomic 来说默认的 存取 操作不仅是原子的而且是有序的。

但是对现代的 CPU 来说 保证顺序一致性的代价是很高昂的。在实践中，编译器为了保证一致性很可能在每次存储操作时都触发 full-blown 内存栅栏。不过如果你的代码可以允许顺序错乱，也就是说你只关注原子性，而不关注顺序。在本例来说就是你可以允许出现 37, 0 的结果，你可以这样写代码：

```C++
           Global
           atomic<int> x, y;

Thread 1                            Thread 2
x.store(17,memory_order_relaxed);   cout << y.load(memory_order_relaxed) << " ";
y.store(37,memory_order_relaxed);   cout << x.load(memory_order_relaxed) << endl;
```

你的 CPU 越现代（modern），这段代码的执行顺序相比上一个就越快。

最后，如果你希望保证访问的顺序性，你可以这样写代码：

```C++
           Global
           atomic<int> x, y;

Thread 1                            Thread 2
x.store(17,memory_order_release);   cout << y.load(memory_order_acquire) << " ";
y.store(37,memory_order_release);   cout << x.load(memory_order_acquire) << endl;
```

这段代码用最小的代价保证访问的顺序性（37， 0 的结果不会在出现）。（在这个例子中和 full-blown 内存栅栏没区别，但是随着程序规模的增长，结果就会出现变化）

如果你只想要 0， 0 或者 37， 17 的结果，你可以用 mutex 把代码包裹起来。既然你已经读到这儿了，我打赌你已经明白原因了。而且本回答长度也快超出我的预期了。（如果用 mutex，也就不用使用 atomic 了）。

基本来说 mutex 这类工具是伟大的发明， C++11 对他们进行了标准化。有时出于性能的考虑你会使用一些底层原语 （例如 [DCL 模式](http://www.justsoftwaresolutions.co.uk/threading/multithreading-in-c++0x-part-6-double-checked-locking.html)）。新标准不仅提供了像 mutexes， condition variables 这些高级工具，也提供了 atomic 和 各种 内存 栅栏这些低层的工具。所以你依据标准就可以写出复杂,高效的并发程序。而且你可以确保你的代码不仅可以在现在的系统上进行编译运行，同样也可以在未来的机器上编译运行。

>不过坦率的说，除非你是这方面的专家并且编写的代码有特别强烈的需求，我建议你坚持使用 mutexes and condition variables.
