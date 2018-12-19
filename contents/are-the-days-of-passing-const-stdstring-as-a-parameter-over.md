# 使用 const std::string& 作为参数进行参数传递的时代已经过去了么？

我最近听了一场 Herb Sutter 的演讲，他建议说 使用 const 引用来传递 std::vector 或者 std::string 的大多数理由已经不成立了。 他建议写类似下面的代码鞥合适：

```C++
std::string do_something ( std::string inval )
{
   std::string return_val;
   // ... do stuff ...
   return return_val;
}
```

我能理解 return_val 在返回点是右值可以通过移动语义来廉价的实现，但是作为参数的 inval 仍然比引用（使用指针实现）要大的多。因为 std::string 有很多成员，比如一个指向堆的指针，还有一个用于短字符串优化的 成员 char[] 。 所以在我看来通过引用来传递仍然是一个好的办法。

谁能解释一下，为什么 Herb 会这么说么？

## 回答

Herb 之所以这么说是因为考虑到如下这种情况：

假设我有一个 函数 A 调用 函数 B，然后 B 调用 函数 C。并且 A 把一个 string 传递给了 B， 然后 B 传递给 C。A 并不知道也不关心 C ，A 仅仅知道 B。也就是说 C 属于 B 的内部实现细节。

假设 A 采用如下方法定义

```C++
void A()
{
  B("value");
}
```

如果 B 和 C 使用 const& 来传递 string 的话，他们看起来应该是下面的样子

```C++
void B(const std::string &str)
{
  C(str);
}

void C(const std::string &str)
{
  //Do something with `str`. Does not store it.
}

```

如你所愿，你仅仅是传递了 指针， 没有拷贝，没有移动，所有人都很满意。 C 使用 const& 形参因为它并不存储这个字符串，它仅仅使用它而已。

现在，我想做一个小小的改变，C 需要在某些地方存放 这个字符串

```C++
void C(const std::string &str)
{
  //Do something with `str`.
  m_str = str;
}
```

嗯？ 构造函数和潜在的内存分配（忽略 [Short String Optimization (SSO)](https://stackoverflow.com/questions/10315041/meaning-of-acronym-sso-in-the-context-of-stdstring)) C++11 的移动语义就是用来消除不需要的拷贝构造函数，不是么？ 并且 A 传递了一个临时变量，C 没有理由再去复制一份数据了，对不对？它把传入的数据占为己有不就可以了么？

但是他不能，因为它得到的是 const 引用。

如果我把 C 改为通过 值 来传递，这只是让 B 在 传递实参时生成拷贝，并不能带来什么实质的改变。

所以，如果我在所有的函数调用层次上都是使用值来传递的话，依赖 std::move 来传递数据，我们就不会上述问题了。如果有人想保留这个数据，随意就好，如果不像，也没什么。

这样做的代价会更高么？ 是的。 移动一个数据比传递引用代价更高。相比拷贝代价更低么？考虑到使用了 SSO 的短字符串，他值得么？

这完全依赖于你的使用场景，你有多么痛恨内存分配？
