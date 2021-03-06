# 动态库和静态库有何区别？

## 回答

动态库是 .so (或者 windows 上的 .dll OS X 上的 .dylib) 文件。所有和这个库相关的代码都在这个文件中，他们在运行期被程序引用。程序通过引用来使用动态库中的代码。

静态库是所有的 .a (Windows 系统 .lib) 文件。所有和这个库相关的代码都在这个文件中， 它在编译过程中被直接链入程序中。程序通过拷贝静态库中的代码到自己内部来实现对静态库代码的使用。（Windows 同样存在用于引用 dll 的 lib 文件）

每一种方法都有优缺点

动态库可以减少每一个程序使用库时需要复制的代码量，可以降低可执行程序的文件大小。它允许你替换具有相同功但是可能性能更高的库，而不需要重新编译可执行程序。但是，共享库在执行函数时会产生一些额外的成本以及运行时加载成本，因为库中的所有符号都需要连接到它们使用的东西。动态库在运行时进行加载，这是实现二进制插件系统的通用机制。

静态库会增加可执行程序文件的大小，但这也意味着你在运行可执行程序时不再依赖那些库的副本。因为所有代码都是编译期链接的，所以在运行期不会产生额外的开销，代码就在那里。

就我个人而言，我倾向于使用动态库，但是当需要保证可执行文件不依赖一些很难满足的依赖时，比如特殊版本的 C++ 标准库或者特定版本的 boost 库，我会使用静态库。
