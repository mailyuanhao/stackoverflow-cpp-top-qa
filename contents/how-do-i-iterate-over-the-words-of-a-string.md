# 如果遍历字符串中的单词 （words）

假设字符串是由空格分割的单词 （words) 组成的。

*对使用 C 风格字符串操作函数或者类似基于字符操作、访问的方案不感兴趣,* *请提供尽量优雅，效率其次的方案。*

我目前已知的最佳实现如下

```cpp
#include <iostream>
#include <sstream>
#include <string>

using namespace std;

int main()
{
    string s = "Somewhere down the road";
    istringstream iss(s);

    do
    {
        string subs;
        iss >> subs;
        cout << "Substring: " << subs << endl;
    } while (iss);
}
```

还有比它更优雅的实现么？

## 问题高票评论

在我看来优雅 （Elegance) 是 **高效漂亮** 的花式叫法。不要拒绝使用那些调用了 C 风格函数的快速灵活的解决方案，仅仅因为它们没有使用模板 (template) 。

## 回答

### 最目前高票答案

两个版本，区别在于存储结果的容器的传递方式

```cpp
#include <string>
#include <sstream>
#include <vector>
#include <iterator>

template<typename Out>
void split(const std::string &s, char delim, Out result) {
    std::stringstream ss(s);
    std::string item;
    while (std::getline(ss, item, delim)) {
        *(result++) = item;
    }
}

std::vector<std::string> split(const std::string &s, char delim) {
    std::vector<std::string> elems;
    split(s, delim, std::back_inserter(elems));
    return elems;
}
```

注意这个解决方案并没有忽略空字符串，下面这段代码会拆分成四个字符串，其中一个是空的。

```cpp
std::vector<std::string> x = split("one:two::three", ':');
```

### 次高票答案

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <algorithm>
#include <iterator>

int main() {
    using namespace std;
    string sentence = "And I feel fine...";
    istringstream iss(sentence);
    copy(istream_iterator<string>(iss),
         istream_iterator<string>(),
         ostream_iterator<string>(cout, "\n"));
}
```

把结果插入容器的版本：

```cpp
vector<string> tokens;
copy(istream_iterator<string>(iss),
     istream_iterator<string>(),
     back_inserter(tokens));
```

直接原地构建容器的版本：

```cpp
vector<string> tokens{istream_iterator<string>{iss},
                      istream_iterator<string>{}};
```

#### 该回答的高票评论

    这个方案不能自定义分隔符，可扩展性和可维护性很差。

