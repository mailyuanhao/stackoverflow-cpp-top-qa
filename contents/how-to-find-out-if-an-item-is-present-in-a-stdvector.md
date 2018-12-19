# 如何确定一个成员是否在 std::vector 中？

## 回答

你可以使用 &lt;algorithm&gt; 中的 [std::find](http://en.cppreference.com/w/cpp/algorithm/find) :

```C++
std::find(vector.begin(), vector.end(), item) != vector.end()
```

返回 bool 类型， 如果存在则返回 true， 不存在返回 false。