# 子类中如何调用基类的函数？

## 回答

I'll take the risk of stating the obvious: 调用就可以了，如果他在基类中被定义，那么它在子类中也是可以直接调用的（除非它是私有的）。

如果子类中有相同签名的函数（覆写），你可以通过在基类名后面加两个冒号来作区分： base_class::foo(...) 。 你要注意，并不像 Java， C#， C++ 没有一个关键字来代表 “基类”（super 或者 base),因为 C++ 支持 [多重继承](http://en.wikipedia.org/wiki/Multiple_inheritance) 这会引起歧义

```C++
class left {
public:
    void foo();
};

class right {
public:
    void foo();
};

class bottom : public left, public right {
public:
    void foo()
    {
        //base::foo();// ambiguous
        left::foo();
        right::foo();

        // and when foo() is not called for 'this':
        bottom b;
        b.left::foo();  // calls b.foo() from 'left'
        b.right::foo();  // call b.foo() from 'right'
    }
};
```

顺便说一下，你不能直接从相同的类同时继承两次因为你将无法从它俩中作区分

```C++
class bottom : public left, public left { // Illegal
};
```
