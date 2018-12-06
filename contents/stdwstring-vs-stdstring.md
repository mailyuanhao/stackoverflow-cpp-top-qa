# std::wstring VS std::string

我无法理解 std::string 和 std::wstring 有什么区别。我知道 wstring 能支持宽字符比如 Unicode 字符，但是我还有以下几个疑问

1. 我什么时候要使用 std::wstring 代替 std::string?
2. std::string 能处理所有的 ASCII 字符么？包括所有的特殊字符？
3. std::wstring 被所有主流 C++ 编译器支持么？
4. 到底什么是 *wide charactoer*？

## 回答

### string ？ wstring ？

std::string 是 [basic::string](http://en.cppreference.com/w/cpp/string/basic_string) 使用 char 的特例化，std::wstring 是使用的 wchar_t.

### char vs. wchar_t

*char* 应该用于保存一个 字符， 通常是一个 8-bit 字符。

*wchar_t* 应该用于保存一个宽字符，事情变得棘手了。

在 Linux 系统上， wchar_t 是 4 bytes 宽，而 Windows 系统上 则是 2 bytes 宽。

### 那么[Unicode](http://en.wikipedia.org/wiki/Unicode) 是什么？

char 和 wchar_t 都不直接和 Unicode 相关联。

#### Linux 平台

拿一个 Linux 系统来说 （我使用的 ubuntu 系统已经支持 Unicode了). 当我处理 char string 时，它原生是使用 [UTF-8](http://en.wikipedia.org/wiki/UTF-8) (比如以 Unicode 编码的字符序列) 编码的。 下面的代码

```C++
#include <cstring>
#include <iostream>

int main(int argc, char* argv[])
{
   const char text[] = "olé" ;


   std::cout << "sizeof(char)    : " << sizeof(char) << std::endl ;
   std::cout << "text            : " << text << std::endl ;
   std::cout << "sizeof(text)    : " << sizeof(text) << std::endl ;
   std::cout << "strlen(text)    : " << strlen(text) << std::endl ;

   std::cout << "text(ordinals)  :" ;

   for(size_t i = 0, iMax = strlen(text); i < iMax; ++i)
   {
      std::cout << " " << static_cast<unsigned int>(
                              static_cast<unsigned char>(text[i])
                          );
   }

   std::cout << std::endl << std::endl ;

   // - - - 

   const wchar_t wtext[] = L"olé" ;

   std::cout << "sizeof(wchar_t) : " << sizeof(wchar_t) << std::endl ;
   //std::cout << "wtext           : " << wtext << std::endl ; <- error
   std::cout << "wtext           : UNABLE TO CONVERT NATIVELY." << std::endl ;
   std::wcout << L"wtext           : " << wtext << std::endl;

   std::cout << "sizeof(wtext)   : " << sizeof(wtext) << std::endl ;
   std::cout << "wcslen(wtext)   : " << wcslen(wtext) << std::endl ;

   std::cout << "wtext(ordinals) :" ;

   for(size_t i = 0, iMax = wcslen(wtext); i < iMax; ++i)
   {
      std::cout << " " << static_cast<unsigned int>(
                              static_cast<unsigned short>(wtext[i])
                              );
   }

   std::cout << std::endl << std::endl ;

   return 0;
}
```

会产生如下输出

```C++
sizeof(char)    : 1
text            : olé
sizeof(text)    : 5
strlen(text)    : 4
text(ordinals)  : 111 108 195 169

sizeof(wchar_t) : 4
wtext           : UNABLE TO CONVERT NATIVELY.
wtext           : ol�
sizeof(wtext)   : 16
wcslen(wtext)   : 3
wtext(ordinals) : 111 108 233
```

可以看出来 文本 "olé" 实际上是由 4 个 char 组成的 110， 108, 195 和 169 （不包含结尾的 0）。（作为练习，请自行研究 wchar_t 版本)

所以，当你在 Linux 系统上使用 char 时，你通常已经在使用 Unicode 了 即使你还没意识到。 同时既然 std::string 内部使用的是 char ，所以 std::string 同样是支持 Unicode 的。

**注意** std::string ，和 C 字符串 API 类似， 会认为  "olé" 包含 四个字符而不是三个，所以你在截断/处理 Unicode 字符时要格外注意，因为很多种字符组合在 UTF-8 中是不允许的。

#### Windows 平台

在 Windows 系统上有一点不同。 Win32 需要支持大量在 Unicode 出现之前就已经存在的产品，而这些产品内部使用 char 并且使用不同的 [代码页](http://en.wikipedia.org/wiki/Code_page)。

所以他们使用了一个很有趣的解决方案。如果一个程序使用 char， 那么 char 字符串会 使用 机器的 本地 代码页 来进行编码或者打印或者显示在 GUI 的标签上。比如 "olé" 在 设置 loacal 为 法语 的机器上会是 "olé" ， 但是在 local 设置为 西里尔 的机器上就不一样了。（如果你使用 [Windows-1251](http://en.wikipedia.org/wiki/Windows-1251) 会是 "olй" ）。这样这些”古老“ 程序依然可以按照它们原有的方式进行工作。

对于基于 Unicode 的程序来说， Windows 使用 2 字节宽 的 wchar_t 并且 使用 [UTF-16](http://en.wikipedia.org/wiki/UTF-16) 进行编码（或者至少是 最兼容 的 UCS-2 编码， 如果我没记错的话）(or at the very least, the mostly compatible UCS-2, which is almost the same thing IIRC)。

使用 char 的程序 被叫做 ”multibyte“ (因为每一个文字都是由一个或者多个 char 组成的)， 同时 使用 wchar_t 的程序被叫做 ”widechar" (每一个文字是有一个或者两个 wchar_t 组成)。参见 Win32 的转换函数  [MultiByteToWideChar](https://msdn.microsoft.com/en-us/library/dd319072.aspx) 和 [WideCharToMultiByte](https://msdn.microsoft.com/en-us/library/dd374130.aspx)。

因此，如果你在 Windows 上工作，你最好使用 wchar-t（除非你使用的框架 比如 GTK+ 或者 Qt 隐藏了这个细节）。事实是，Windows 本身使用的就是wchar-t， 所以即使是使用 char 的 ”古老“ 程序在调用比如 SetWindowText() 这些 API 时， Windows 内部也会把 char 转换为 wchar-t。

#### 内存占用问题？

UTF-32 每个字符使用 4 个字节，通常来说 UTF-8 文本和 UTF-16 文本比 UTF-32 使用更少的内存， 没有什么要额外说明的。

如果有内存占用问题，你要知道对于大多数西方语言来说， UTF-8 会比相同的 UTF-16 占用更少的内存。

对于其他语言来说（比如中文，日文），内存占用基本上差不多，可能 UTF-8 会比 UTF-16 要多一点。

总的来说， UTF-16 通常使用 2 个字节偶尔使用 4 个字节 除非你需要处理一些深奥的语言（比如 Klingon? Elvish?）。UTF-8 会根据情况使用 1-4 个字节。

参考 [链接](http://en.wikipedia.org/wiki/UTF-8#Compared_to_UTF-16) 获取更多的信息。

### 结论

1. 什么情况下要使用 std::wstring 来替代 std::string ?
    1. Linux 系统上永远不要
    2. Windows 系统上，一直这样做就好了
    3. 跨平台的的代码，依赖你使用的工具库
    4. 对于 第一第二点，还要参考你使用的 工具库和框架库 的建议
2. std::string 能处理所有的 ASCII 字符么？包括特殊字符？  
_注意  std::string 可以用于处理二进制信息，而 std::wstring 则不可以_  
Linux 系统？ 是的  
Windows 系统？ 仅能处理当前用户的 locale 的特殊字符  
std::string 可以处理所有基于 char 的字符串（每个 char 是一个 0-255 的数字） 但是：
    1. ASCII 应该是从 0 到 127. 更高的 char 并不属于 ASCII 字符集。
    2. 从 0 到 127 的 char 可以被正确的处理
    3. 从 128 到 255 的 char 依赖于你使用的编码（Unicode 或者 非 Unicode），但是如果使用 UTF-8 编码则可以处理所有的 Unicode 文字。
3. 主流的 C++ 编译器都支持 std::wstring 么？  
绝大多数是得。除了移植到 Windows 系统上的 基于 GCC 的编译器。  
在我的 g++ 4.3.2（Linux系统）上可以正常工作，而且从 Visual C++ 6 开始我就一直使用 std::wstring
4. 到底什么是 宽字符 wide character？  
在 C、C++ 里，它是一种比 char 字符类型更大的字符类型 写作：wchar_t。它应该被用于存储字符索引（indice）（比如 Unicode 文字）大于 255（或者 127） 的字符。