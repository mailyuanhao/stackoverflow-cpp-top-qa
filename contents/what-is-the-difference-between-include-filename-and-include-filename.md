# #include <filename> 和 #include "filename" 的区别是什么？

## 回答

在实践中，区别在于预处理器寻找包含文件的位置不同。

方式 #include <filename> 寻找方式依赖于实现。通常来说是在编译器、IDE的预定义路径中搜索。通常用于包含标准库头文件。

方式 #include "filename"， 先在包含该预处理指示的源文件所在的路径中搜索，然后在再 #include <filename> 方式进行搜索。该种方案主要用于包含程序自定义头文件。

完整的描述可以在 GCC 对应的[文档](https://gcc.gnu.org/onlinedocs/cpp/Search-Path.html)中查看

参考 [C 标准](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=182) 6.10.2。 其实基本都是依赖于实现的。具体编译器具体分析，最高票答案回答的在基本上就是实践中通行的做法。