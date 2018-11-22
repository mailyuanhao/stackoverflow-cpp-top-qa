# 未定义的引用，未找到的外部符号 错误是什么意思？ 怎么解决？

undefined reference, unresolved external symbol

## 回答

C++ 程序的编译过程分为多个步骤， [(credits to Keith Thompson for the reference)](https://stackoverflow.com/a/8834196/673730) 2.2 中进行了规定。

步骤过于专业，有些术语不知道怎么翻译。就不翻译了。直接搬过来原文。其实这个错误发生在链接阶段，只要知道编译过程大致分成 预处理，编译和链接三个步骤就能理解这篇答案了。

>The precedence among the syntax rules of translation is specified by the following phases [see footnote].

Physical source file characters are mapped, in an implementation-defined manner, to the basic source character set (introducing new-line characters for end-of-line indicators) if necessary. [SNIP]
Each instance of a backslash character (\) immediately followed by a new-line character is deleted, splicing physical source lines to form logical source lines. [SNIP]
The source file is decomposed into preprocessing tokens (2.5) and sequences of white-space characters (including comments). [SNIP]
Preprocessing directives are executed, macro invocations are expanded, and _Pragma unary operator expressions are executed. [SNIP]
Each source character set member in a character literal or a string literal, as well as each escape sequence and universal-character-name in a character literal or a non-raw string literal, is converted to the corresponding member of the execution character set; [SNIP]
Adjacent string literal tokens are concatenated.
White-space characters separating tokens are no longer significant. Each preprocessing token is converted into a token. (2.7). The resulting tokens are syntactically and semantically analyzed and translated as a translation unit. [SNIP]
Translated translation units and instantiation units are combined as follows: [SNIP]
All external entity references are resolved. Library components are linked to satisfy external references to entities not defined in the current translation. All such translator output is collected into a program image which contains information needed for execution in its execution environment. (emphasis mine)
[footnote] Implementations must behave as if these separate phases occur, although in practice different phases might be folded together.


问题中的错误发生在编译的最后一个阶段，也就是通常说的链接阶段。链接基本上来说就是，你已经把一堆实现文件编译成目标文件或者静态库了，现在想把他们组合在一起工作。

加入你在 a.cpp 中定义了 一个符号 a。 现在 b.cpp 中声明并且使用了这个符号。 链接开始之前， 简单的认为这个符号已经在别处定义了，至于在哪儿定义的目前并不关心。 链接过程负责找到这个符号然后正确的把它连接到 b.cpp 上（实际上是目标文件或者静态库）。

如果你使用 Microsoft Visual Studio， 你就会发现 工程 会生成 .lib 文件。这些文件包含导出符号表和导入符号表。 导入符号表从链接目标中进行决议， 导出符号表提供给使用本 .lib 的 库使用。

别的编译器，平台 也存在类似的机制。

Microsoft Visual Studio 会产生 如 **error LNK2001, error LNK1120, error LNK2019** 这样的错误信息， 而 GCC 通常产生类似 **undefined reference to 符号名** 的错误信息。

代码

```C++
struct X
{
   virtual void foo();
};
struct Y : X
{
   void foo() {}
};
struct A
{
   virtual ~A() = 0;
};
struct B: A
{
   virtual ~B(){}
};
extern int x;
void foo();
int main()
{
   x = 0;
   foo();
   Y y;
   B b;
}
```
GCC 会产生如下的错误码

>/home/AbiSfw/ccvvuHoX.o: In function `main':
prog.cpp:(.text+0x10): undefined reference to `x'
prog.cpp:(.text+0x19): undefined reference to `foo()'
prog.cpp:(.text+0x2d): undefined reference to `A::~A()'
/home/AbiSfw/ccvvuHoX.o: In function `B::~B()':
prog.cpp:(.text._ZN1BD1Ev[B::~B()]+0xb): undefined reference to `A::~A()'
/home/AbiSfw/ccvvuHoX.o: In function `B::~B()':
prog.cpp:(.text._ZN1BD0Ev[B::~B()]+0x12): undefined reference to `A::~A()'
/home/AbiSfw/ccvvuHoX.o:(.rodata._ZTI1Y[typeinfo for Y]+0x8): undefined reference to `typeinfo for X'
/home/AbiSfw/ccvvuHoX.o:(.rodata._ZTI1B[typeinfo for B]+0x8): undefined reference to `typeinfo for A'
collect2: ld returned 1 exit status

而 Mic Visu Studio 则产生如下错误信息

>1>test2.obj : error LNK2001: unresolved external symbol "void __cdecl foo(void)" (?foo@@YAXXZ)
1>test2.obj : error LNK2001: unresolved external symbol "int x" (?x@@3HA)
1>test2.obj : error LNK2001: unresolved external symbol "public: virtual __thiscall A::~A(void)" (??1A@@UAE@XZ)
1>test2.obj : error LNK2001: unresolved external symbol "public: virtual void __thiscall X::foo(void)" (?foo@X@@UAEXXZ)
1>...\test2.exe : fatal error LNK1120: 4 unresolved externals

常见原因如下

- [Failure to link against appropriate libraries/object files or compile implementation files](https://stackoverflow.com/a/12574400/673730)
- [Declared and undefined variable or function.](https://stackoverflow.com/a/12574403/673730)
- [Common issues with class-type members](https://stackoverflow.com/a/12574407/673730)
- [Template implementations not visible.](https://stackoverflow.com/a/12574417/673730)
- [Symbols were defined in a C program and used in C++ code.](https://stackoverflow.com/a/12574420/673730)
- [Incorrectly importing/exporting methods/classes across modules/dll. (MSVS specific)](https://stackoverflow.com/a/12574423/673730)
- [Circular library dependency](https://stackoverflow.com/a/20358542/673730) 
- [undefined reference to `WinMain@16'](https://stackoverflow.com/questions/5259714/undefined-reference-to-winmain16/5260237#5260237) 
- [Interdependent library order](https://stackoverflow.com/a/24675715/1356926) 
- [Multiple source files of the same name](https://stackoverflow.com/questions/14364362/visualstudio-project-with-multiple-sourcefiles-of-the-same-name) 
- [Mistyping or not including the .lib extension when using the #pragma (Microsoft Visual Studio)](https://stackoverflow.com/a/25744263/3747990) 
- [Problems with template friends](https://stackoverflow.com/a/35891188/3747990) 
- [Inconsistent UNICODE definitions](https://stackoverflow.com/a/36475406/3747990)