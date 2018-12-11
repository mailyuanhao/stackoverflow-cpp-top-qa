# 如何把 std::string 和 int 连接起来？

我原以为这是很简单的事情，但是遇到了很多困难。 如果我有如下变量

```C++
std::string name = "John";
int age = 21;
```

我如何把他们连接起来来获得一个单独的字符串 "John21" ?

## 回答

按字母顺序排列：

```C++
std::string name = "John";
int age = 21;
std::string result;

// 1. with Boost
result = name + boost::lexical_cast<std::string>(age);

// 2. with C++11
result = name + std::to_string(age);

// 3. with FastFormat.Format
fastformat::fmt(result, "{0}{1}", name, age);

// 4. with FastFormat.Write
fastformat::write(result, name, age);

// 5. with the {fmt} library
result = fmt::format("{}{}", name, age);

// 6. with IOStreams
std::stringstream sstm;
sstm << name << age;
result = sstm.str();

// 7. with itoa
char numstr[21]; // enough to hold all numbers up to 64-bits
result = name + itoa(age, numstr, 10);

// 8. with sprintf
char numstr[21]; // enough to hold all numbers up to 64-bits
sprintf(numstr, "%d", age);
result = name + numstr;

// 9. with STLSoft's integer_to_string
char numstr[21]; // enough to hold all numbers up to 64-bits
result = name + stlsoft::integer_to_string(numstr, 21, age);

// 10. with STLSoft's winstl::int_to_string()
result = name + winstl::int_to_string(age);

// 11. With Poco NumberFormatter
result = name + Poco::NumberFormatter().format(age);
```

1. 安全的，但是效率低； 需要 [Boost](http://www.boost.org/)（仅需要头文件）；绝大多数/全部 平台支持
2. 安全的； 需要 C++11 （[to_string()](http://www.cplusplus.com/reference/string/to_string/) 已经包含在 string 头文件中)
3. 安全；效率高； 需要 [FastFormat](http://fastformat.sourceforge.net/) 库； 需要编译； 大多数/全部 平台；
4. 安全；效率高； 需要 [FastFormat](http://fastformat.sourceforge.net/) 库； 需要编译； 大多数/全部 平台；
5. 安全；效率高；需要 [the {fmt} library](https://github.com/fmtlib/fmt), 可以编译也可以纯头文件使用； 大多数/全部平台
6. 安全；效率低而且繁琐；需要 sstream 头文件（标准库提供）
7. 容易出问题（你需要提供足够大的缓冲区）；效率高，繁琐； itoa() 属于非标准的扩展，并不保证全平台都支持
8. 容易出问题（你必须提供足够大的缓冲区）；效率高，繁琐；没有额外依赖（属于标准 C++）； 全平台
9. 容易出问题（你必须提供足够大的缓冲区）；[可能是最快的转换方式](http://www.ddj.com/cpp/184401596), 繁琐，需要 [STLSoft](http://www.stlsoft.org/) (纯头文件)； 大多数/全 平台
10. safe-ish (不要在单条语句中进行多次 int_to_string() 调用), 效率高; 需要 [STLSoft](http://www.stlsoft.org/) (纯头文件); 仅仅支持 Windows 平台
11. 安全；效率低；需要 [Poco C++](http://www.boost.org/); 大多数/全 平台
