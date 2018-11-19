# < 比 <= 快么？

## 回答

不会，在绝大多数架构上 < 并不会比 <= 快。 在 x86 架构上所有比较操作（integral comparisons）通常都是通过执行两条 机器指令 实现的

1. 设置 EFLAGS 的 test 或者 cmp 指令
2. 和 [Jcc (jump) instruction](http://www.unixwiz.net/techtips/x86-jumps.html) 指令（根据比较类型和代码布局可能是下面之一）
    1. jne - Jump if not equal --> ZF = 0
    2. jz - Jump if zero (equal) --> ZF = 1
    3. jg - Jump if greater --> ZF = 0 and SF = OF
    4. 其他类似的跳转指令

---

### 例子

```C++
    if (a < b) {
        // Do something 1
    }

```

使用如下命令编译

```sh
gcc -m32 -S -masm=intel test.c
```

生成如下代码

```nasm
    mov     eax, DWORD PTR [esp+24]      ; a
    cmp     eax, DWORD PTR [esp+28]      ; b
    jge     .L2                          ; jump if a is >= b
    ; Do something 1
.L2:
```

而

```C++
    if (a <= b) {
        // Do something 2
    }
```

会生成如下代码

```nasm
    mov     eax, DWORD PTR [esp+24]      ; a
    cmp     eax, DWORD PTR [esp+28]      ; b
    jg      .L5                          ; jump if a is > b
    ; Do something 2
.L5:
```

区别仅仅在于 jg 和 jge， 而这两条指令的耗时是相同的。

---

（这一段是详细说明 Jcc 各指令消耗时间是否相同的，大体意思就是 虽然 没有地方说明这些指令消耗的时间相同，但是在 Intel 的文档中这些指令都是捆绑在一块介绍的（不管是指令说明，还是在优化章节中）。所以 intel 的文档也没说 Jcc 各指令中某个指令与众不同。  建议看原文，翻译就别看了。我不懂。。。。）。

我写这个评论是为了指出 没有任何迹象能说明 不同的 jmp 指令消耗相同的时间。这个问题有点棘手，下面是我能给出的：在 [Intel Instruction Set Reference](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html), 中，这些被组织在同一个 通用指令（common instruction） Jcc (Jump if condition is met) 里面。在 [Optimization Reference Manual](http://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-optimization-manual.html) 附录 C 中他们也是按相同的分组方式组织的。

>Latency — The number of clock cycles that are required for the execution core to complete the execution of all of the μops that form an instruction. 执行核心运行一条指令所有的微指令所需要的时钟周期。  
Throughput — The number of clock cycles required to wait before the issue ports are free to accept the same instruction again. For many instructions, the throughput of an instruction can be significantly less than its latency 发射端可以接受相同的指令之前需要等待的时钟周期数。多大多数指令来说，throughput 可以明显低于 它的 latency。

（太深入的知识了，按我的理解是 CPU 核心内部是有流水线执行的，所以 一条指令消耗的时钟周期 要 大于 CPU 核心 接受下一条相同指令的时间。）

Jcc 的 Latency 和 Throughput 如下：

```C++
      Latency   Throughput
Jcc     N/A        0.5
```

而 Jcc 在文档中有如下脚注

>7) Selection of conditional jump instructions should be based on the recommendation of section Section 3.4.1, “Branch Prediction Optimization,” to improve the predictability of branches. When branches are predicted successfully, the latency of jcc is effectively zero.

所以在 Intel 的文档中， 并没有任何地方把 Jcc 下的某一条指令单独拿出来说它于其它 Jcc 指令不同。

如果从指令的具体实现电路上去考虑，可以假设 EFLAGS 的 bit 位上有简单的 与或门 来判断某种条件是否成立。没有理由说检测两个 bit 位的指令消耗的时间会比检测一个 bit 位的指令消耗的时间 长 或者 短。 (Ignoring gate propagation delay, which is much less than the clock period.)

---

### 浮点数

对于 x87 的浮点运算来说，上述结论同样适用（和上面段落的源码相同，只不过 double 替换了 int ）

```nasm
        fld     QWORD PTR [esp+32]
        fld     QWORD PTR [esp+40]
        fucomip st, st(1)              ; Compare ST(0) and ST(1), and set CF, PF, ZF in EFLAGS
        fstp    st(0)
        seta    al                     ; Set al if above (CF=0 and ZF=0).
        test    al, al
        je      .L2
        ; Do something 1
.L2:

        fld     QWORD PTR [esp+32]
        fld     QWORD PTR [esp+40]
        fucomip st, st(1)              ; (same thing as above)
        fstp    st(0)
        setae   al                     ; Set al if above or equal (CF=0).
        test    al, al
        je      .L5
        ; Do something 2
.L5:
        leave
        ret
```