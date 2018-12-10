# 类成员函数声明时 最后的 “const” 关键字是什么意思？

## 回答

当你给成员方法添加 const  关键字 时，this 指针会变成指向 const 对象的指针，所以你不能修改任何数据成员（除非你用 mutalbe 修饰该数据成员，后面会详细介绍）

const 关键字属于方法标示的一部分，也就是说你可以创建两个类似的方法，一个是 const 对象时调用的，另一个则是非 const 对象调用的。

```C++
#include <iostream>

class MyClass
{
private:
    int counter;
public:
    void Foo()
    { 
        std::cout << "Foo" << std::endl;    
    }

    void Foo() const
    {
        std::cout << "Foo const" << std::endl;
    }

};

int main()
{
    MyClass cc;
    const MyClass& ccc = cc;
    cc.Foo();
    ccc.Foo();
}
```

会有如下的输出

```sh
Foo
Foo const
```

在 非 const 版本里面你可以修改实例的成员，而在 const 版本里面则不允许。 如果你把上面代码中的 方法修改成如下的样子，你将遇到编译错误

```C++
    void Foo()
    {
        counter++; //this works
        std::cout << "Foo" << std::endl;    
    }

    void Foo() const
    {
        counter++; //this will not compile
        std::cout << "Foo const" << std::endl;
    }
```

上面的说法并不完全正确，因为你可以把某个成员声明为 mutable 这样的话 一个 const 方法就可以修改它了。这一般用在内部计数器之类的东西上。上面问题的解决方法如下

```C++
#include <iostream>

class MyClass
{
private:
    mutable int counter;
public:

    MyClass() : counter(0) {}

    void Foo()
    {
        counter++;
        std::cout << "Foo" << std::endl;    
    }

    void Foo() const
    {
        counter++;
        std::cout << "Foo const" << std::endl;
    }

    int GetInvocations() const
    {
        return counter;
    }
};

int main(void)
{
    MyClass cc;
    const MyClass& ccc = cc;
    cc.Foo();
    ccc.Foo();
    std::cout << "The MyClass instance has been invoked " << ccc.GetInvocations() << " times" << endl;
}
```

上述代码会有如下输出

```sh
Foo
Foo const
The MyClass instance has been invoked 2 times
```

