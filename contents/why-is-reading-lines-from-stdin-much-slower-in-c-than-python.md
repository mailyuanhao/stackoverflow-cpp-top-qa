# 为什么从标准输入读取数据， C++ 要比 Python 慢很多？

C++ 代码

```C++
#include <iostream>
#include <time.h>

using namespace std;

int main() {
    string input_line;
    long line_count = 0;
    time_t start = time(NULL);
    int sec;
    int lps;

    while (cin) {
        getline(cin, input_line);
        if (!cin.eof())
            line_count++;
    };

    sec = (int) time(NULL) - start;
    cerr << "Read " << line_count << " lines in " << sec << " seconds.";
    if (sec > 0) {
        lps = line_count / sec;
        cerr << " LPS: " << lps << endl;
    } else
        cerr << endl;
    return 0;
}

// Compiled with:
// g++ -O3 -o readline_test_cpp foo.cpp
```

Python 代码

```python
#!/usr/bin/env python
import time
import sys

count = 0
start = time.time()

for line in  sys.stdin:
    count += 1

delta_sec = int(time.time() - start_time)
if delta_sec >= 0:
    lines_per_sec = int(round(count/delta_sec))
    print("Read {0} lines in {1} seconds. LPS: {2}".format(count, delta_sec,
       lines_per_sec))
```

测试结果

```sh
$cat test_lines | ./readline_test_cpp
Read 5570000 lines in 9 seconds. LPS: 618889

$cat test_lines | ./readline_test.py
Read 5570000 lines in 1 seconds. LPS: 5570000
```

## 回答

默认情况下 cin 和 stdio 是同步的，中间没有缓冲区，如果你在代码开头加上下面的代码，性能会得到极大的改善

```C++
std::ios_base::sync_with_stdio(false);
```

通常情况下一个带缓冲的输入流，每次会读取一大块数据，而不是一个个字符(character)去读取。这会减少相对来说开销较大的系统调用次数。 但是同是基于 FILE× 实现的 stdio 和 iostreams 通常是独立实现的，而且有各自的缓冲区，这会在同时使用二者时引发问题。比如：

```C++
int myvalue1;
cin >> myvalue1;
int myvalue2;
scanf("%d",&myvalue2);
```

如果 cin 读取了超过它需要的数据，会造成 scanf 无法读取到有效的数据，因为它们使用了不同的缓冲区。从而引发无法预料的结果。

为了避免这种情况， cin 默认是和 stdio 同步的。一种通常的实现方式是 cin 调用 stdio 的相关函数 按照需要逐字（character）读取数据，不幸的是，这种方法会产生大量额外的开销。对于少量数据的读取，这些额外开销还不明显，如果要读取百万行的数据，这些额外的开销就会很显著了。

所幸，标准库的作者提供了关闭这个特性的[方法](http://en.cppreference.com/w/cpp/io/ios_base/sync_with_stdio)，在充分了解后果的情况下，可以关闭这个特性来提供性能。