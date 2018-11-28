# 怎么把 std::string 转换为 const char * 或者 char* ？

## 回答

如果你只是想把 [std::string](http://en.cppreference.com/w/cpp/string/basic_string) 传递给 一个需要 const char* 参数的函数，你可以这样做：

```C++
std::string str;
const char * c = str.c_str();
```

如果你想得到一个可写的拷贝，比如说 char* ，你可以这样做

```C++
std::string str;
char * writable = new char[str.size() + 1];
std::copy(str.begin(), str.end(), writable);
writable[str.size()] = '\0'; // don't forget the terminating 0

// don't forget to free the string after finished using it
delete[] writable;
```

**注意**, 上述代码并不是异常安全的，如果在 new 和 delete 之间的任意代码发生了异常，就会出现内存泄露。可以有两种方法解决这个问题

### boost::scoped_array

[boost::scoped_array](http://www.boost.org/doc/libs/release/libs/smart_ptr/scoped_array.htm) 会在作用域结束时自动释放内存

```C++
std::string str;
boost::scoped_array<char> writable(new char[str.size() + 1]);
std::copy(str.begin(), str.end(), writable.get());
writable[str.size()] = '\0'; // don't forget the terminating 0

// get the char* using writable.get()

// memory is automatically freed if the smart pointer goes 
// out of scope
```

### std::vector

这是标准库的方法，不用依赖外部库。你可以使用 [std::vector](http://en.cppreference.com/w/cpp/container/vector)来自动管理内存。

```C++
std::string str;
std::vector<char> writable(str.begin(), str.end());
writable.push_back('\0');

// get the char* using &writable[0] or &*writable.begin()
```
