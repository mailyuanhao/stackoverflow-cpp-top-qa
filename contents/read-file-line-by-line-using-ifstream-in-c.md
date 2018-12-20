# 在 C++ 中使用 ifstream 逐行读取文件

文件内容如下

>5 3  
6 4  
7 1  
10 5  
11 6  
12 3  
12 4

5 3 是一个坐标，我在 C++ 中怎么逐行的处理这些数据？

## 回答

首先 创建 ifstream 实例

```C++
#include <fstream>
std::ifstream infile("thefile.txt")
```

有两种标准的方法：

1. 假设每一行都由两个数字组成可以逐个 token 的读取：

```C++
int a, b;
while(infile >> a >> b)
{
    //process pair (a, b)
}
```

2. 基于行的解析，使用字符串流

```C++
#include <sstream>
#include <string>

std::string line;
while (std::getline(infile, line))
{
    std::istringstream iss(line);
    int a, b;
    if (!(iss >> a >> b)) { break; } // error

    // process pair (a,b)
}
```

你不能混合使用 (1) 和 (2), 因为基于 token 的解析并不读入换行符，所以你在它后面使用 getline() 会得到一个空行，因为 基于 token 的解析已经到达行尾了。
