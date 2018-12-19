# 怎样把一个 vector 追加到 另一个 vector 后面？

## 回答

```C++
a.insert(a.end(), b.begin(), b.end());
```

或者

```C++
a.insert(std::end(a), std::begin(b), std::end(b));
```

第二种方案更加通用，b 甚至可以是数组，但是你需要 C++ 11;
