# C++ 中 struct 和 typedef struct 有什么区别？

## 回答

在 C++ 中它们仅有细微的区别。这是从 C 中延续过来的，在 C 中是有区别的。

C 语音 标准([C89 §3.1.2.3](http://port70.net/~nsz/c/c89/c89-draft.txt), [C99 §6.2.3](http://port70.net/~nsz/c/c99/n1256.html#6.2.3), 和 [C11 §6.2.3](http://port70.net/~nsz/c/c11/n1570.html#6.2.3)) 要求不同类型的标识符要在不同的命名空间中， 包括 tag identifers （例如 struct/union/enum) 和 ordinary identifiers (例如 typedef 和其他标识符)。

如果你仅仅写

```C++
struct Foo { ... };
Foo x;
```

你将无法通过编译，因为 Foo 仅仅在 tag namespace 中被定义。

你需要这样声明：

```C++
struct Foo x;
```

任何时候你想使用 Foo， 你都要这样写 struct Foo，这很快就会让人厌烦，所以你可以添加一个 typedef：

```C++
struct Foo { ... };
typedef struct Foo Foo;
```

现在 struct Foo （在 tag 命名空间）和 普通的 Foo （在 ordinary identifer 命名空间）全部指向同一个事物，在定义对象时你就可以直接写 Foo 而不用加上 struct 关键字了。

---

```C++
typedef struct Foo { ... } Foo;
```

仅仅是声明和 typedef 在一块的一种缩写方式。

---

最终

```C++
typedef struct { ... } Foo;
```

声明了一个匿名的结构体同时给他创建一个 typedef。使用这种写法，该结构体就仅在 typedef 命名空间有名称，在 tag 命名空间中是没有名字的。这意味着它也不能做前置声明了。如果你想使用前置声明，你就应该给他一个在 tag 命名空间的名字。

---

在 C++ 中 所有的 struct / union / enum/ class 声明表现的像默认 做 typedef 一样， 只要该名字没有被其他 一样的名字 隐藏。 