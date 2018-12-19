# 如何把一个 ASCII 编码的文件整个读取到 std::string 中？

## 回答

> 注意该方法虽然很符合 C++ STL 惯用法，但是性能很低！ 不要在大文件上使用该方法（参考  http://insanecoding.blogspot.com/2011/11/how-to-read-in-file-in-c.html）

你可以基于这个文件构建一个 streambuf iterator, 然后通过它来初始化字符串

```C++
#include <string>
#include <fstream>
#include <streambuf>

std::ifstream t("file.txt");
std::string str((std::istreambuf_iterator<char>(t)),
                 std::istreambuf_iterator<char>());
```

请注意第一个参数被额外的小括号包括，这是必须的，这可以避免 "[most vexing parse](http://web.archive.org/web/20110426155617/http://www.informit.com/guides/content.aspx?g=cplusplus&seqNum=439)" 问题，这次这种情况下并不会像通常那样引发编译错误，而是给出有趣（错误）的结果。

下面是预先分配内存的版本，不在依赖 string 内部的自动重新分配机制

```C++
#include <string>
#include <fstream>
#include <streambuf>

std::ifstream t("file.txt");
std::string str;

t.seekg(0, std::ios::end);   
str.reserve(t.tellg());
t.seekg(0, std::ios::beg);

str.assign((std::istreambuf_iterator<char>(t)),
            std::istreambuf_iterator<char>());
```
