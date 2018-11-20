# 在 C++ 中 从 int 到 stirng 转换哪种方法最简单？

我知道两种方法，还有更简单的么？

```C++
int a = 10;
char *intStr = itoa(a);
string str = string(intStr);
```

```C++
int a = 10;
stringstream ss;
ss << a;
string str = ss.str();
```

## 回答

C++11 引入了 [std::stoi](http://en.cppreference.com/w/cpp/string/basic_string/stol)(每种数值类型都提供了对应的方法)和 [std::to_string](http://en.cppreference.com/w/cpp/string/basic_string/to_string),对应了 C 中的 atoi 和 itoa， 只不过是使用 std::string 表述的

```C++
#include <string> 

std::string s = std::to_string(42);
```

这是据我所知最简单的方法。你甚至可以忽略类型声明，使用 auto 关键字

```C++
auto s = std::to_string(42);
```
