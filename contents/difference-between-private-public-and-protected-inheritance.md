# private，protected 和 public 继承的区别

# 回答

为了回答这个问题，我想先用我自己的方式来描述成员的访问标号。如果你已经了解了，请跳过，从 next 开始。

据我所知有三种访问标号： public private protected

```C++
class Base {
    public:
        int publicMember;
    protected:
        int protectedMember;
    private:
        int privateMember;
};
```

任何能察觉到 Base 的事物，同样能察觉到 Base 包含 publicMember

仅有 Base 的子类或者子类的子类，能察觉到 Base 的 protectedMember

除了 Base 本身，谁都无法察觉到 privateMember

“察觉” 的意思是 “知道它的存在，而且能够访问”

### next

同样的情况也出现在 public， protected 和 private 继承上。让我们考虑一个场景 有个 Base 类 和他的子类 Child

如果是 public 继承，所有能察觉到 Base 和 Child 的同样能察觉到 Child 从 Base 继承。

如果是 protected 继承，仅有 Child 和他的 子类能察觉到自己是继承至 Base 的。
de
如果是 private 继承，仅有 Child 能察觉到 自己是继承至 Base 的。

**注意， C++ 的可见性是基于 类 而不是 对象的。 也就是说同一个类的对象可以互相访问对方的数据，不存在任何限制。**