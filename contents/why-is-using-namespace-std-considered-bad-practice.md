# 为什么 using namespace std 被认为是不好的做法？

有人告诉我在代码中写 using namespace std 是错误的，应该直接使用 std::cout, std::cin 来代替。

为什么这是不好的做法？它会影响性能么？

## 回答

和性能没有一点关系。考虑如下情况， 你同时使用 Foo 和 Bar 两个库：

```C++
using namespace Foo;
using namespace Bar;
```

你调用 Foo 中的 Blah() 和 Bar 中的 Quux() 这一点问题没有。直到有一天你把 Foo 升级到 新版本 2.0， 它也提供了一个名字叫 Quux() 的函数。 因为 Foo 2.0 和 Bar 同时把 Quxx() 导入到全局命名空间， 这就产生了冲突。为了解决冲突要花费一定的代价，尤其是两个函数的参数也匹配的时候。

如果恰巧 Foo 2.0 引入的 Quux() 能明确的更好的匹配你之前对 Bar::Quux() 的调用， 事情会更加的糟糕。你的代码仍然能通过编译，但是没有人知道它调用了错误的 Quux() 函数。

记住 std 命名空间有大量的名称 (identifiers)， 有很多通用的命名比如 list, sort， string， iterator 等，很可能在其它库中出现。

如果你之前就使用 Foo::Blah() 和 Bar::Quux()， 那么 Foo::Quux() 的引入不会造成任何问题。

## 译者注

考虑下面这段代码

```C++
#include <iostream>
#include <cstdint>

namespace Foo{
void Blah(){
        std::cout << "Foo::Blah" << std::endl;
}
/*
void Quxx(int32_t){
        std::cout << "Foo::Quxx" << std::endl;
}
*/
}

namespace Bar{
void Quxx(uint32_t){
        std::cout << "Bar::Quxx" << std::endl;
}
}

using namespace Foo;
using namespace Bar;

int main() {
        Quxx(32);
        return 0;
}
```

在 Foo 引入 Quxx() 之前， 调用的是 Bar::Quxx() 但是一旦 Foo 引入了 Quxx() 则会默默的转而调用 Foo::Quxx() 因为做参数类型匹配时 Bar::Quxx() 需要做一次整形提升操作， 而匹配 Foo::Quxx() 则不需要。
