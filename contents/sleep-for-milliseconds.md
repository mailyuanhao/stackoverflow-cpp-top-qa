# 怎么实现 sleep 几毫秒？

## 回答

C++11，你可以使用标准库来实现

```C++
#include <chrono>
#include <thread>
std::this_thread::sleep_for(std::chrono::milliseconds(x));
```

清晰可读，再也不用去猜测 sleep 函数的单位了。
