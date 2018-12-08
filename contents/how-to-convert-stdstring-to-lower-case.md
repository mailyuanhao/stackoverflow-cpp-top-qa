# 如何把 std::string 转换为 小写？

## 回答1

参考[该文章](http://notfaq.wordpress.com/2007/08/04/cc-convert-string-to-upperlower-case/)

```C++
#include <algorithm>
#include <string> 

std::string data = "Abc"; 
std::transform(data.begin(), data.end(), data.begin(), ::tolower);
```

遍历每一个字符是无法避免的，没有方法能预知某个字符是大小还是小写的。

如果你实在不想使用 tolower(), 下面是一个无法通用的一个实现版本，并不建议你使用。

```C++
char easytolower(char in) {
  if(in <= 'Z' && in >= 'A')
    return in - ('Z' - 'z');
  return in;
}

std::transform(data.begin(), data.end(), data.begin(), easytolower);
```

注意： ::tolower() 只能做 单个 byte 字符的替换，对部分语言的文字来说并不适合，尤其是使用多字节编码时，比如 UTF-8.

### comment
- 这种方式使用 ::tolower 可能触发 crash，比如传入 -1. 因为对于 非 ASCII 的输入 是 未定义行为。

## 回答2

Boost string algorithm 有先关的方法

```C++
#include <boost/algorithm/string.hpp>    

std::string str = "HELLO, WORLD!";
boost::algorithm::to_lower(str); // modifies str

#include <boost/algorithm/string.hpp>    

const std::string str = "HELLO, WORLD!";
const std::string lower_str = boost::algorithm::to_lower_copy(str);
```
