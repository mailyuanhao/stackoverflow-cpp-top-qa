# 为什么要使用 static_cast &lt int &gt (x) 来代替 (int)x ？

## 回答

最重要的原因是使用 C 风格的类型转换并不能有效的区分我们称之为 static_cast<>(), reinterpret_cast<>(), const_cast<>(), 和 dynamic_cast<>() 等的转换。这四种类型是完全不一样的。

static_cast<>() 通常来说是安全的，因为这说明要么语言层面上这种转换是有效的，要么有一个合适的构造函数能支持这种转换。唯一可能会出现风险的场景是，你做由继承树上层到下层的转换时，做这种转换需要你自己保证被转换的对象确实属于转换目标类或者其后代类，也就是说你需要语言层面之外的机制来实现这种保证的判断（比如在对象内部放置标记成员量）。这种情况使用 dynamic_cast<>() 是安全的只要你检查转换的结果（如果转换对象是指针的话），或者会抛出转换失败的异常（转换引用时）。

与之相反使用 reinterpret_cast<>() (或者 const_cast<>()) 总是危险的。这相当于你告诉编译器： “相信我，虽然它看起来不像是 foo （看起来是个不变量），但它确实是”。

使用 C 风格类型转换的第一个问题是，如果你不理解所有的转换细节或者不去读大量的上下文代码，你就无法快速准确的确定到底进行了哪一种转换。

让我们假设如下场景

```C++
class CMyClass : public CMyBase {...};
class CMyOtherStuff {...} ;

CMyBase  *pSomething; // filled somewhere
```

现在使用相同的方式编译下面的代码

```C++
CMyClass *pMyObject;
pMyObject = static_cast<CMyClass*>(pSomething); // Safe; as long as we checked

pMyObject = (CMyClass*)(pSomething); // Same as static_cast<>
                                     // Safe; as long as we checked
                                     // but harder to read
```

不过，下面两个几乎一致的代码

```C++
CMyOtherStuff *pOther;
pOther = static_cast<CMyOtherStuff*>(pSomething); // Compiler error: Can't convert

pOther = (CMyOtherStuff*)(pSomething);            // No compiler error.
                                                  // Same as reinterpret_cast<>
                                                  // and it's wrong!!!
```

在不了解所有涉及到的 类 的情况下你无法简单的区分这两种情况。

第二个问题是， C 风格类型转换很难在代码中定位。在复杂的表达式中很难在其中看到哪儿发生了 C 风格的转换（In complex expressions it can be very hard to see C-style casts.）。 在不引入完整的的 C++ 编译器前端的前提下，几乎不可能写一个自动化的工具来定位所有的 C 风格类型转换。 从另一方面说，搜索 "static_cast<" 或者 "reinterpret_cast<" 确很容易。

```C++
pOther = reinterpret_cast<CMyOtherStuff*>(pSomething);
      // No compiler error.
      // but the presence of a reinterpret_cast<> is
      // like a Siren with Red Flashing Lights in your code.
      // The mere typing of it should cause you to feel VERY uncomfortable.
```

总的来说：C 风格的类型转换不仅更加危险，而且很难定位到所有发生转换的地方来确认他们都是正确的。
