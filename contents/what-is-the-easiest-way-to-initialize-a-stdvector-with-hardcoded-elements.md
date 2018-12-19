# 使用硬编码数据初始化 std::vector 的最简单的额方法是？

数组可以使用如下的方式初始化：

```C++
int a[] = {10, 20, 30};
```

有同样优雅的方法来初始化 std::vector 么？

我知道最好的方法是：

```C++
std::vector<int> ints;

int.push_back(10);
int.push_back(20);
int.push_back(30);
```

还有更好的方法么？

## 回答1

如果你的编译器支持 C++ 11， 你可以简单的这么做：

```C++
std::vector<int> v = {1, 2, 3}
```

或者使用 [Boost Assign](http://www.boost.org/doc/libs/1_42_0/libs/assign/doc/index.html);

```C++
#include <boost/assign/list_of.hpp>
...
std::vector<int> v = boost::assign::list_of(1)(2)(3)(4);
```

或者

```C++
#include <boost/assign/std/vector.hpp>
using namespace boost::assign;
...
std::vector<int> v;
v += 1, 2, 3, 4;
```
