# C++11 中的 using 和 typedef 有什么区别？

我知道从 C++11 开始我们可以使用 using 来定义别名，就像 typedef 做的

```C++
typedef int MyInt;
```

现在我们可以使用 using

```C++
using MyInt = int;
```

新的语法使得我们能够表达 "template typedef":

```C++
template< class T > using MyType = AnotherType< T, MyAllocatorType >;
```

但是对于前两个非模板的例子，他们在标准中有什么细微的差别么？ 比如 typedef 定义的别名仅仅是个新的名字，而没有定义新的类型。（它们之间的转换都是隐式执行的）。这个说法适用于 using 么？ 它定义新的类型么？ 他们之间到底有没有区别？

## 问题评论

我更倾向于使用新的语法，因为它特别像普通的变量赋值语法，可读性更高。比如 后面两种写法你喜欢哪一种？ typedef void (&MyFunc)(int,int); 或者 using MyFunc = void(int,int); 

## 回答

在标准中它们是等价的
>A typedef-name can also be introduced by an alias-declaration. The identifier following the using keyword becomes a typedef-name and the optional attribute-specifier-seq following the identifier appertains to that typedef-name. **It has the same semantics as if it were introduced by the typedef specifier**. In particular, it does not define a new type and it shall not appear in the type-id.

typedef 名字 也可以使用 别名声明 方式引入。 using 关键字后面的标识符会成为一个 typedef 名字 并且 标识符 后可选的 attribute—specifer-seq 也附属于该 typedef 名字。它拥有的语义就如同他是通过 typedef 引入的。 特别是它并不定义新类型，也不允许出现在 type-id 中。