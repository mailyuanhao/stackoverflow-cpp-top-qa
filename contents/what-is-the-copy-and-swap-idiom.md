# copy-and-swap 惯用法

这个惯用法是什么意思？什么时候使用？能解决什么问题？在 C++11 中该惯用法是否需要变动？

## 回答

### 总览

任何需要管理资源的类（封装类，类似智能指针）都需要实现 [The Big Three](https://stackoverflow.com/questions/4172722/what-is-the-rule-of-three)。相比之下拷贝构造函数和析构函数的实现和作用比较直观，赋值操作符的实现是这三个中最困难的也是最需要细致考虑的。需要怎么实现它？实现过程中有哪些坑需要注意？

copy-and-swap 惯用法是该问题的一个解决方案，他优雅的解决了赋值操作符实现过程中的两个问题：避免 [代码复制粘贴](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 和 提供 [强异常安全保证](http://en.wikipedia.org/wiki/Exception_guarantees)。

### 它是如何实现的？

大体上说，它首先利用拷贝构造函数创建目标对象的一个临时的拷贝，其次 利用 swap 函数 交换原对象和该临时拷贝的数据，最后临时对象析构时销毁原来的旧数据。通过上述三个步骤达到对象复制的目的。

要完成上述事情，需要三个前提，一个能工作的拷贝构造函数，一个能工作的析构函数（资源管理类，无论如何都要实现这函数功能），以及一个 swap 函数。

swap 是一个通过交换每个成员数据来实现交换某个类的两个对象数据的 non-throwing 函数。我们并不能使用 std::swap 来代替，因为 std::swap 内部依赖类的拷贝构造和拷贝赋值函数，而我们的目的是为了实现拷贝赋值函数。

### 更深层的阐述

#### 目标

考虑一个场景，我们要实现一个管理动态数组的类。首先要编写构造，拷贝构造和析构函数：

```C++
#include <algorithm> // std::copy
#include <cstddef> // std::size_t

class dumb_array
{
public:
    // (default) constructor
    dumb_array(std::size_t size = 0)
        : mSize(size),
          mArray(mSize ? new int[mSize]() : nullptr)
    {
    }

    // copy-constructor
    dumb_array(const dumb_array& other)
        : mSize(other.mSize),
          mArray(mSize ? new int[mSize] : nullptr),
    {
        // note that this is non-throwing, because of the data
        // types being used; more attention to detail with regards
        // to exceptions must be given in a more general case, however
        std::copy(other.mArray, other.mArray + mSize, mArray);
    }

    // destructor
    ~dumb_array()
    {
        delete [] mArray;
    }

private:
    std::size_t mSize;
    int* mArray;
};
```

要完善这个类的功能，还欠缺 operator= 的实现。

### 糟糕的方案

下面例子展示一种糟糕的实现：

```C++
// the hard part
dumb_array& operator=(const dumb_array& other)
{
    if (this != &other) // (1)
    {
        // get rid of the old data...
        delete [] mArray; // (2)
        mArray = nullptr; // (2) *(see footnote for rationale)

        // ...and put in the new
        mSize = other.mSize; // (3)
        mArray = mSize ? new int[mSize] : nullptr; // (3)
        std::copy(other.mArray, other.mArray + mSize, mArray); // (3)
    }

    return *this;
}
```

到这儿，可以说功能完成了。被管理的动态数组不会出现内存泄露问题。但是仍然有三个问题，依次如下：

1. 检查是不是自身赋值。这个检查一是为了避免执行不必要的操作，另一个是避免出现一些微妙的 bug （比如删除了要被复制的原数组）。其余的情况下它只会影响程序的性能，而且这个检查也影响代码的整洁度，毕竟自我赋值是极少出现的情况。如果 operator= 的实现能够不需要这个检查，就更好了。
2. 另一个是它只提供了基本异常安全保证。如果 new int[mSize] 抛出异常，被赋值对象的数据就被修改了（换句话说， mSize 值不对了，而且源数据被销毁了）。要提供强异常安全的保证，需要作如下的修改：
```C++
dumb_array& operator=(const dumb_array& other)
{
    if (this != &other) // (1)
    {
        // get the new data ready before we replace the old
        std::size_t newSize = other.mSize;
        int* newArray = newSize ? new int[newSize]() : nullptr; // (3)
        std::copy(other.mArray, other.mArray + newSize, newArray); // (3)

        // replace the old data (all are non-throwing)
        delete [] mArray;
        mSize = newSize;
        mArray = newArray;
    }

    return *this;
}
```

3. 代码膨胀。这会导致第三个问题，重复的代码。operator = 实现代码是其他地方代码的重复。这是很严重的事情。

在这个例子里，重复代码核心只有两行（分配内存和数据拷贝），但是处理更复杂的资源时，代码膨胀就会变得相当麻烦。我们应该坚持永远不要重复自己的原则。

（也许有些人会问，这么多代码只是为了管理一个资源，如果资源变多了怎么办？这种担心其实是毫无必要的，因为[永远不要在一个类中管理操作一个资源](http://en.wikipedia.org/wiki/Single_responsibility_principle)）。

### 有效方案

上面提到过， copy-and-swap 惯用法会解决上述的问题。现在只欠缺 swap 方法的实现了。The Rule of Three 原则 要求管理资源的类必须实现 构造函数，拷贝构造函数和析构函数，其实这个原则应该修改成  "The Big Three and A Half" 意即：在任何时候只要一个类负责管理资源，那么就要提供 swap 函数。

swap 函数添加如下

```C++
class dumb_array
{
public:
    // ...

    friend void swap(dumb_array& first, dumb_array& second) // nothrow
    {
        // enable ADL (not necessary in our case, but good practice)
        using std::swap;

        // by swapping the members of two objects,
        // the two objects are effectively swapped
        swap(first.mSize, second.mSize);
        swap(first.mArray, second.mArray);
    }

    // ...
};
```

swap 声明为 public friend 的原因[参见](https://stackoverflow.com/questions/5695548/public-friend-swap-member-function)

使用 swap 函数通常来说性能会更高，在本例中 swap 仅仅交换了指针和 size 的内容，省去了分配和拷贝的操作。有了 swap 函数后，我们具备了实现 copy-and-swap 惯用法的所有前提条件。

赋值操作符可以简单的做如下实现：

```C++
dumb_array& operator=(dumb_array other) // (1)
{
    swap(*this, other); // (2)

    return *this;
}
```

仅仅做一处改变就可以一次性解决上述的三个问题。

### 工作原理

先关注一处重要的抉择：参数使用值传递。实际上有些人会简单的选择下面的实现方式（实际上很多糟糕的 copy-and-swap 惯用法确实这么实现的）

```C++
dumb_array& operator=(const dumb_array& other)
{
    dumb_array temp(other);
    swap(*this, temp);

    return *this;
}
```

这会让我们的代码丧失一次[性能优化机会](https://web.archive.org/web/20140113221447/http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/)。不仅如此，后面会讨论为什么这个选择在 C++11 中是至关重要的。（重要提示：如果你想在一个函数中复制某个值，那么让编译器在参数列表中做复制，也就是值传递）。

不论上面哪个选择，都可以消除重复的代码，因为资源获取可以直接复用拷贝构造函数。现在已经获取了资源，下面进行 swap

请注意，在函数进入时，新的数据就已经准备好了（分配，拷贝完毕）。这相当于让我们的代码免费获得了强异常安全保证，因为如果拷贝构造失败，函数根本不会执行修改 *this 的代码。我们之前需要手动去做这个保证，现在编译器直接替我们做了。

剩下的事情就轻而易举了，在无异常的情况下，简单的交换源数据和临时数据。源数据被交换到临时对象中，在该临时对象生命期结束时，自动得到释放。

本惯用法的赋值操作符的实现不会重复代码，也就不会引入新的 bug。同时新的实现方法不可能出现自我赋值的可能，对应的检查也就可以免了。

这就是 copy-and-swap 惯用法的意思。

### C++11带来的影响

在 C++11 中有一处重要的改变会影响资源管理的方式。the Rule of Three 应该叫 The Rule of Four (and a half)。因为我们不仅可以拷贝构造我们的资源，现在还需要支持[移动语义](https://stackoverflow.com/questions/3106110/can-someone-please-explain-move-semantics-to-me)。

很幸运，我们可以简单的修改如下

```C++
class dumb_array
{
public:
    // ...

    // move constructor
    dumb_array(dumb_array&& other)
        : dumb_array() // initialize via default constructor, C++11 only
    {
        swap(*this, other);
    }

    // ...
};
```

这段代码做了什么？请回忆一下移动构造函数的目的：从另一个对象中拿到资源，并保证被拿走资源的对象处于能被正常销货或者重新赋值的状态。（to take the resources from another instance of the class, leaving it in a state guaranteed to be assignable and destructible.）。

代码做的事情很简单，利用默认构造函数进行构造（C++11 的特性），然后和 other 对象交换资源。因为默认构造函数构造的对象肯定处于正常状态（能被赋值或者销毁），所以被交换的 other 的状态也是正常的。

### 他为什么能工作？

仅仅做了一个改变，为什么他能正常工作？ 回想之前的提到的抉择：采用传值而不是传引用的方式：

```C++
dumb_array& operator=(dumb_array other); // (1)
```

如果 other 是被一个右值初始化的，他将会被移动构造。同样的如果是 C++03 传值的方式可以复用拷贝构造的代码， C++11 会在适用的时候直接进行移动构造。（参考前面提到的文章，拷贝、移动都可以被编译器优化掉）。

上述就是 copy-and-swap 惯用法。

