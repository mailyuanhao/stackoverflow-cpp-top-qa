# nullptr 到底是什么？

C++11 给我们带来了很多新的特性。至少对我来说 新的 nullptr 很有意思也让我感到迷惑。

再也不用使用丑陋的 NULL 宏了。

```C++
int* x = nullptr;
myclass* obj = nullptr;
```

不过，我仍然不理解 nullptr 宏到底怎么工作的，比如 [Wikipedia 文章](http://en.wikipedia.org/wiki/C%2B%2B11#Null_pointer_constant) 说

> C++11 通过引入一个新的关键字 nullptr 来当作一个区分空指针的常量。它属于 能够隐式转换成（也可以直接作比较）任意指针类型或者指向成员的指针类型 的 nullptr_t 类型。他不能隐式转换成（或者比较）整形类型，除了 bool。

它怎么能既是一个关键字也可以是一种类型的实例呢？

并且，你能否给出另一个例子（除了 Wikipedia 那个）展示 nullptr 比 旧 的 0 更好的地方么？

## 注释

[integral type](https://en.cppreference.com/w/cpp/types/is_integral)

## 回答

> 它怎么能既是一个关键字也可以是一种类型的实例呢？

这并不奇怪，true 和 false 也都既是关键字也是 bool 类型实例的字面量。nullptr 是 std::nullptr_t 类型的指针实例字面量，并且它是右值（你不能使用 & 来取它的地址）。

- 4.10 指针转换相关内容说，std::nullptr_t 的右值类型的实例属于空指针常量，一个整形的空指针常量可以转换为 std::nullptr_t， 反过来转换是不允许的。你可以使用指针和整形来进行函数重载，传递 nullptr 会选择接收指针的重载函数版本。传递 NULL 或者 0 会让人迷惑的选择 int 型的版本。
- nullptr_t 需要使用 reinterpret_cast 来转换为 整形类型，并且和 (void*)0 这种转换具有相同的语义 (mapping implementation defined)。 reinterpret_cast 无法把 nullptr_t 转换为任何指针类型，如果可以的话可以使用隐式类型转换或者使用 static_cast;
- 标准要求 sizeof(nullptr_t) 等同于 sizeof(void*);
  