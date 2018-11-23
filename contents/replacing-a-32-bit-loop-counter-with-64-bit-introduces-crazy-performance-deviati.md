# 循环计数器由 32bit 替换为 64bit 产生巨大的性能差异

## 问题

我在寻找 对大量数据进行 popcount 运算的最快方法时遇到了非常奇怪的现象。把循环变量 由 unsigned 修改为 uint64_t 后，在我的机器上性能下降了 50%. 基准测试代码如下

```C++
#include <iostream>
#include <chrono>
#include <x86intrin.h>

int main(int argc, char* argv[]) {

    using namespace std;
    if (argc != 2) {
       cerr << "usage: array_size in MB" << endl;
       return -1;
    }

    uint64_t size = atol(argv[1])<<20;
    uint64_t* buffer = new uint64_t[size/8];
    char* charbuffer = reinterpret_cast<char*>(buffer);
    for (unsigned i=0; i<size; ++i)
        charbuffer[i] = rand()%256;

    uint64_t count,duration;
    chrono::time_point<chrono::system_clock> startP,endP;
    {
        startP = chrono::system_clock::now();
        count = 0;
        for( unsigned k = 0; k < 10000; k++){
            // Tight unrolled loop with unsigned
            for (unsigned i=0; i<size/8; i+=4) {
                count += _mm_popcnt_u64(buffer[i]);
                count += _mm_popcnt_u64(buffer[i+1]);
                count += _mm_popcnt_u64(buffer[i+2]);
                count += _mm_popcnt_u64(buffer[i+3]);
            }
        }
        endP = chrono::system_clock::now();
        duration = chrono::duration_cast<std::chrono::nanoseconds>(endP-startP).count();
        cout << "unsigned\t" << count << '\t' << (duration/1.0E9) << " sec \t"
             << (10000.0*size)/(duration) << " GB/s" << endl;
    }
    {
        startP = chrono::system_clock::now();
        count=0;
        for( unsigned k = 0; k < 10000; k++){
            // Tight unrolled loop with uint64_t
            for (uint64_t i=0;i<size/8;i+=4) {
                count += _mm_popcnt_u64(buffer[i]);
                count += _mm_popcnt_u64(buffer[i+1]);
                count += _mm_popcnt_u64(buffer[i+2]);
                count += _mm_popcnt_u64(buffer[i+3]);
            }
        }
        endP = chrono::system_clock::now();
        duration = chrono::duration_cast<std::chrono::nanoseconds>(endP-startP).count();
        cout << "uint64_t\t"  << count << '\t' << (duration/1.0E9) << " sec \t"
             << (10000.0*size)/(duration) << " GB/s" << endl;
    }

    free(charbuffer);
}
```

如你所见，我们创建一个拥有 x 兆字节随机数的数组， x 由命令行传入。 然后遍历数组使用一个 展开的 x86 popcount 指令 对数据进行popcount 运算。为了得到精确的结果，我们进行了 10,000 遍的 popcount 运算。然后测量耗时。前面使用 unsigned 作为 循环变量，后面使用 uint64_t 作为循环变量。我感觉这点变化不应该有影响，但是结果却大相径庭。

### （令人瞠目结舌的）结果

我使用如下命令进行编译（g++ version: Ubuntu 4.8.2-19ubuntu1）：

```C++
g++ -O3 -march=native -std=c++11 test.cpp -o test
```

下面是在我的 [Haswell Core i7-4770K](http://en.wikipedia.org/wiki/Haswell_%28microarchitecture%29#Desktop_processors) CPU @ 3.50 GHz 上运行 **test 1** （也就是 1M 随机数据）的结果：

>unsigned 41959360000 0.401554 sec 26.113 GB/s  
uint64_t 41959360000 0.759822 sec 13.8003 GB/s

如你所见， uint64_t 版本的吞吐量仅仅是 unsigned 版本的一半。问题看起来是生成的汇编指令不同产生的，但是为什么？刚上来我感觉是编译器 bug，随后我测试了 clang++ (Ubuntu Clang version 3.4-1ubuntu3)

```sh
clang++ -O3 -march=native -std=c++11 teest.cpp -o test
```

**test 1** 的结果

>unsigned 41959360000 0.398293 sec 26.3267 GB/s  
uint64_t 41959360000 0.680954 sec 15.3986 GB/s

结果同样令人诧异。但是更令人诧异的还在后面，我把 buffer 大小改为常量：

```C++
uint64_t size = atol(argv[1]) << 20;
```

到：

```C++
uint64_t size = 1 << 20;
```

修改后编译器在编译期间就能知道 buffer 的大小。也许这可以让它进行一些优化，下面是 g++ 的结果：

>unsigned 41959360000 0.509156 sec 20.5944 GB/s  
uint64_t 41959360000 0.508673 sec 20.6139 GB/s

现在两个版本的性能基本一致了。但是 unsigned 版本变得更慢了。从 26 降低到 20 GB/s，也就是说从非 常量改为常量反而进行了负优化。严格的说，对于这个现象我完全没有头绪。下面是clang++ 的结果：

>unsigned 41959360000 0.677009 sec 15.4884 GB/  
uint64_t 41959360000 0.676909 sec 15.4906 GB/s

等等，怎么会？ 心在两个版本同时降到了 15GB/s。对于 clang++ 来说仅仅把一个非 常量 改为 常量 造成两个版本都降低了性能。

我让一个同事在他 [Ivy Bridge](http://en.wikipedia.org/wiki/Ivy_Bridge_%28microarchitecture%29) CPU 上编译运行我的测试代码。他的机器上结果差不多。这个问题看起来和 Haswell 无关，因为结果都很令人奇怪，看起来也不时编译器 bug。 我们手头并没有 AMD CPU，所以无法测试 AMD CPU 的结果。

### 更加疯狂的结果

同样用第一个例子的代码（使用 atol(argv[1]) 的版本）， 然后我们在变量前增加 static 关键字。

```C++
static uint64_t size=atol(argv[1])<<20;
```

下面是 g++ 的结果：

>unsigned 41959360000 0.396728 sec 26.4306 GB/s  
uint64_t 41959360000 0.509484 sec 20.5811 GB/s

是的，还有另一种结果，u32 版本达到了 26 GB/s 的速度，但是 u64 版本 的速度从 13 GB/s 提升到了 20 GB/s !。*在我同事的机器上，u64 版本的速度测得了最快的值，甚至比 u32 版本还快！*.不幸的是，这个结果仅适用于 g++, clang++ 看起来并不受 static 的影响。

### 我的疑问

你解答下面三个问题么？

1. 为什么 u32 和 u64 的版本差距那么大？
2. 为什么 把 非常量 替换为 常量 产生了负优化版本的代码？
3. 为什么 static 关键字 可以让 u64 版本跑的更快？甚至比我同事机器上最初的版本还快。

我知道 优化 本身就很诡异，但是我没想到如此小的改变会产生完全不一样的执行时间，像修改常量 buffer 大小这种小影响会再一次让结果更加混乱。当然，我想一直都能得到最快 26 GB/s 的结果。但是对于这种情况目前我能想到的最可靠的办法就是 拷贝粘贴 汇编代码，然后使用内敛汇编实现。这是我目前唯一能避免编译器因为小小的改变就会发疯的方法。你怎么想？还有其它能稳定获得最高性能的方法么？

### 反汇编

下面上上面各种版本的反汇编结果

26 GB/s version from g++ / u32 / non-const bufsize:

```nasm
0x400af8:
lea 0x1(%rdx),%eax
popcnt (%rbx,%rax,8),%r9
lea 0x2(%rdx),%edi
popcnt (%rbx,%rcx,8),%rax
lea 0x3(%rdx),%esi
add %r9,%rax
popcnt (%rbx,%rdi,8),%rcx
add $0x4,%edx
add %rcx,%rax
popcnt (%rbx,%rsi,8),%rcx
add %rcx,%rax
mov %edx,%ecx
add %rax,%r14
cmp %rbp,%rcx
jb 0x400af8
```

13 GB/s version from g++ / u64 / non-const bufsize:

```nasm
0x400c00:
popcnt 0x8(%rbx,%rdx,8),%rcx
popcnt (%rbx,%rdx,8),%rax
add %rcx,%rax
popcnt 0x10(%rbx,%rdx,8),%rcx
add %rcx,%rax
popcnt 0x18(%rbx,%rdx,8),%rcx
add $0x4,%rdx
add %rcx,%rax
add %rax,%r12
cmp %rbp,%rdx
jb 0x400c00
```

15 GB/s version from clang++ / u64 / non-const bufsize:

```nasm
0x400e50:
popcnt (%r15,%rcx,8),%rdx
add %rbx,%rdx
popcnt 0x8(%r15,%rcx,8),%rsi
add %rdx,%rsi
popcnt 0x10(%r15,%rcx,8),%rdx
add %rsi,%rdx
popcnt 0x18(%r15,%rcx,8),%rbx
add %rdx,%rbx
add $0x4,%rcx
cmp %rbp,%rcx
jb 0x400e50
```

20 GB/s version from g++ / u32&u64 / const bufsize:

```nasm
0x400a68:
popcnt (%rbx,%rdx,1),%rax
popcnt 0x8(%rbx,%rdx,1),%rcx
add %rax,%rcx
popcnt 0x10(%rbx,%rdx,1),%rax
add %rax,%rcx
popcnt 0x18(%rbx,%rdx,1),%rsi
add $0x20,%rdx
add %rsi,%rcx
add %rcx,%rbp
cmp $0x100000,%rdx
jne 0x400a68
```

15 GB/s version from clang++ / u32&u64 / const bufsize:

```nasm
0x400dd0:
popcnt (%r14,%rcx,8),%rdx
add %rbx,%rdx
popcnt 0x8(%r14,%rcx,8),%rsi
add %rdx,%rsi
popcnt 0x10(%r14,%rcx,8),%rdx
add %rsi,%rdx
popcnt 0x18(%r14,%rcx,8),%rbx
add %rdx,%rbx
add $0x4,%rcx
cmp $0x20000,%rcx
jb 0x400dd0
```

有意思， 最快的 26GB/s 的版本是最长的。这看起来像是唯一使用 lea 的版本。有一些版本使用 jb 跳转，另一些使用 jne。除了这点差异，每个版本都看起来差不多。我看不出来哪儿会产生这么大的性能差异，但是我不是很擅长分析汇编代码。最慢的 （13 GB/s） 的版本甚至跟起来更短更优。谁能解释一下？

### 得到的教训

不管最终的答案是什么，我都学到了 对于 循环次数很大的循环体来说每一个细节都会有影响，即使这个细节看起来和关键代码没有任何关系。我从来没考虑过要选择适当的循环变量，但是如你所见，一点小更改就产生了这么大的差距。即使是 存储类型都会有巨大的影响，就像你看到的我们在 size 变量前 static 关键字的效果。以后，涉及到对性能要求高的紧凑热循环时，我会尝试在不同的编译器上测试各种可能。

还有一点很奇怪，即使我把循环展开四次，结果还是有很大的差距。所以即使展开后，你也会被结果惊讶道。相当有意思！

## 回答

罪魁祸首： 错误的数据相关性（False Data Dependency） （而且编译器并没有注意到这个问题）

在 Sandy，Ivy Bridge 和 Haswell 处理器上， 指令

```nasm
popcnt src， dest
```

看起来错误的关联了目标寄存器 dest。 即使该指令仅仅写目标寄存器也会在执行前等待目标寄存器 dest 准备好 （ready）

（数据相关是指后面的指令依赖前面指令的结果，这会降低流水线的吞吐量）

这种关联并不仅仅发生在单次循环的 4 个 popcnt 之间，它甚至跨越循环迭代，造成处理器不能并行处理不同的循环迭代。

unsigned vs uint64_t 以及其他的调教都不直接影响这个问题，但是他们会影响用于把寄存器分配给变量的寄存器分配器。

在你的例子里，速度直接取决于 关联链 而这是由寄存器分配器决定的：

- 13 GB/s has a chain: popcnt-add-popcnt-popcnt → next iteration 
- 15 GB/s has a chain: popcnt-add-popcnt-add → next iteration
- 20 GB/s has a chain: popcnt-popcnt → next iteration 
- 26 GB/s has a chain: popcnt-popcnt → next iteration

20 和 26 的差别看起来是由神奇的间接寻址影响的，不管怎么样，在你速度到这个级别时 处理器会碰到各种瓶颈。

---

为了测试，我使用内联汇编来忽略编译器影响直接得到想要的汇编码。我还分开了 count 变量来打断 可能对结果造成影响的 所有其他的 数据关联。

结果如下

Sandy Bridge Xeon @ 3.5 GHz:（完整的代码在最后）

>GCC 4.6.3: g++ popcnt.cpp -std=c++0x -O3 -save-temps -march=native  
Ubuntu 12

不同的寄存器 18.6195 GB/s

```nasm
.L4:
    movq    (%rbx,%rax,8), %r8
    movq    8(%rbx,%rax,8), %r9
    movq    16(%rbx,%rax,8), %r10
    movq    24(%rbx,%rax,8), %r11
    addq    $4, %rax

    popcnt %r8, %r8
    add    %r8, %rdx
    popcnt %r9, %r9
    add    %r9, %rcx
    popcnt %r10, %r10
    add    %r10, %rdi
    popcnt %r11, %r11
    add    %r11, %rsi

    cmpq    $131072, %rax
    jne .L4
```

相同的寄存器 8.49272 GB/s

```nasm
.L9:
    movq    (%rbx,%rdx,8), %r9
    movq    8(%rbx,%rdx,8), %r10
    movq    16(%rbx,%rdx,8), %r11
    movq    24(%rbx,%rdx,8), %rbp
    addq    $4, %rdx

    # This time reuse "rax" for all the popcnts.
    popcnt %r9, %rax
    add    %rax, %rcx
    popcnt %r10, %rax
    add    %rax, %rsi
    popcnt %r11, %rax
    add    %rax, %r8
    popcnt %rbp, %rax
    add    %rax, %rdi

    cmpq    $131072, %rdx
    jne .L9
```

相同寄存器并且打断关联 17.8869 GB/s

```nasm
.L14:
    movq    (%rbx,%rdx,8), %r9
    movq    8(%rbx,%rdx,8), %r10
    movq    16(%rbx,%rdx,8), %r11
    movq    24(%rbx,%rdx,8), %rbp
    addq    $4, %rdx

    # Reuse "rax" for all the popcnts.
    xor    %rax, %rax    # Break the cross-iteration dependency by zeroing "rax".
    popcnt %r9, %rax
    add    %rax, %rcx
    popcnt %r10, %rax
    add    %rax, %rsi
    popcnt %r11, %rax
    add    %rax, %r8
    popcnt %rbp, %rax
    add    %rax, %rdi

    cmpq    $131072, %rdx
    jne .L14
```

### 编译器出现了什么问题？

这看起来不管是 GCC 还是 Visual Studio 都没有注意到 popcnt 有这种错误关联问题。然而这种错误关联并不罕见，这仅仅是编译器有没有注意到的问题。

popcnt 确切来说并不是常用的指令。所以主流的编译器忽略到这一点并不奇怪。而且看起来并没有文档提到这个问题。如果 Intel 不透漏的话，除非有人碰到，外面的人就无法得知了。

（更新： [GCC 4.9.2](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=62011#c13)已经注意到这个错误关联的问题，在激活优化的情况下已经对生成的代码进行处理来弥补这个错误。 其他主流的编译器 如 Clang， MSVC 甚至 Intel 自家的 ICC 都还没有关注到这个微指令界别的问题，所以还没有针对该问题生成补偿代码。

### 为什么 CPU 会出现这种错误关联的问题？

我们只能猜测，不过看起来 Intel 对很多的双操作数指令做了相同的处理。 常见的指令 如 add， sub 把两个操作数都当做输入。所以 Intel 很可能为了简化处理器设计把 popcnt 也归到这一类了。

AMD 的处理器看起来并没有这种错误关联问题。

---

完整的测试代码如下

```C++
#include <iostream>
#include <chrono>
#include <x86intrin.h>

int main(int argc, char* argv[]) {

   using namespace std;
   uint64_t size=1<<20;

   uint64_t* buffer = new uint64_t[size/8];
   char* charbuffer=reinterpret_cast<char*>(buffer);
   for (unsigned i=0;i<size;++i) charbuffer[i]=rand()%256;

   uint64_t count,duration;
   chrono::time_point<chrono::system_clock> startP,endP;
   {
      uint64_t c0 = 0;
      uint64_t c1 = 0;
      uint64_t c2 = 0;
      uint64_t c3 = 0;
      startP = chrono::system_clock::now();
      for( unsigned k = 0; k < 10000; k++){
         for (uint64_t i=0;i<size/8;i+=4) {
            uint64_t r0 = buffer[i + 0];
            uint64_t r1 = buffer[i + 1];
            uint64_t r2 = buffer[i + 2];
            uint64_t r3 = buffer[i + 3];
            __asm__(
                "popcnt %4, %4  \n\t"
                "add %4, %0     \n\t"
                "popcnt %5, %5  \n\t"
                "add %5, %1     \n\t"
                "popcnt %6, %6  \n\t"
                "add %6, %2     \n\t"
                "popcnt %7, %7  \n\t"
                "add %7, %3     \n\t"
                : "+r" (c0), "+r" (c1), "+r" (c2), "+r" (c3)
                : "r"  (r0), "r"  (r1), "r"  (r2), "r"  (r3)
            );
         }
      }
      count = c0 + c1 + c2 + c3;
      endP = chrono::system_clock::now();
      duration=chrono::duration_cast<std::chrono::nanoseconds>(endP-startP).count();
      cout << "No Chain\t" << count << '\t' << (duration/1.0E9) << " sec \t"
            << (10000.0*size)/(duration) << " GB/s" << endl;
   }
   {
      uint64_t c0 = 0;
      uint64_t c1 = 0;
      uint64_t c2 = 0;
      uint64_t c3 = 0;
      startP = chrono::system_clock::now();
      for( unsigned k = 0; k < 10000; k++){
         for (uint64_t i=0;i<size/8;i+=4) {
            uint64_t r0 = buffer[i + 0];
            uint64_t r1 = buffer[i + 1];
            uint64_t r2 = buffer[i + 2];
            uint64_t r3 = buffer[i + 3];
            __asm__(
                "popcnt %4, %%rax   \n\t"
                "add %%rax, %0      \n\t"
                "popcnt %5, %%rax   \n\t"
                "add %%rax, %1      \n\t"
                "popcnt %6, %%rax   \n\t"
                "add %%rax, %2      \n\t"
                "popcnt %7, %%rax   \n\t"
                "add %%rax, %3      \n\t"
                : "+r" (c0), "+r" (c1), "+r" (c2), "+r" (c3)
                : "r"  (r0), "r"  (r1), "r"  (r2), "r"  (r3)
                : "rax"
            );
         }
      }
      count = c0 + c1 + c2 + c3;
      endP = chrono::system_clock::now();
      duration=chrono::duration_cast<std::chrono::nanoseconds>(endP-startP).count();
      cout << "Chain 4   \t"  << count << '\t' << (duration/1.0E9) << " sec \t"
            << (10000.0*size)/(duration) << " GB/s" << endl;
   }
   {
      uint64_t c0 = 0;
      uint64_t c1 = 0;
      uint64_t c2 = 0;
      uint64_t c3 = 0;
      startP = chrono::system_clock::now();
      for( unsigned k = 0; k < 10000; k++){
         for (uint64_t i=0;i<size/8;i+=4) {
            uint64_t r0 = buffer[i + 0];
            uint64_t r1 = buffer[i + 1];
            uint64_t r2 = buffer[i + 2];
            uint64_t r3 = buffer[i + 3];
            __asm__(
                "xor %%rax, %%rax   \n\t"   // <--- Break the chain.
                "popcnt %4, %%rax   \n\t"
                "add %%rax, %0      \n\t"
                "popcnt %5, %%rax   \n\t"
                "add %%rax, %1      \n\t"
                "popcnt %6, %%rax   \n\t"
                "add %%rax, %2      \n\t"
                "popcnt %7, %%rax   \n\t"
                "add %%rax, %3      \n\t"
                : "+r" (c0), "+r" (c1), "+r" (c2), "+r" (c3)
                : "r"  (r0), "r"  (r1), "r"  (r2), "r"  (r3)
                : "rax"
            );
         }
      }
      count = c0 + c1 + c2 + c3;
      endP = chrono::system_clock::now();
      duration=chrono::duration_cast<std::chrono::nanoseconds>(endP-startP).count();
      cout << "Broken Chain\t"  << count << '\t' << (duration/1.0E9) << " sec \t"
            << (10000.0*size)/(duration) << " GB/s" << endl;
   }

   free(charbuffer);
}
```

[这儿](http://pastebin.com/kbzgL8si)有个类似的测试。

那个测试测试了 popcnt 不同 错误关联链。

```C++
False Chain 0:  41959360000 0.57748 sec     18.1578 GB/s
False Chain 1:  41959360000 0.585398 sec    17.9122 GB/s
False Chain 2:  41959360000 0.645483 sec    16.2448 GB/s
False Chain 3:  41959360000 0.929718 sec    11.2784 GB/s
False Chain 4:  41959360000 1.23572 sec     8.48557 GB/s
```