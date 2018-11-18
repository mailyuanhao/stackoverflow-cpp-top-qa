# 什么是智能指针？在什么场景下使用？

## 回答

智能指针是用于封装裸指针并管理裸指针指向资源的生命周期的类。智能指针并不是单一的类型，不过所有的类型的实现都是对裸指针的抽象。

相对于裸指针应该优先选择使用智能指针。如果你确定必须要使用指针，那么就要考虑使用智能指针，因为使用它们可以缓解使用裸指针带来的很多问题，比如因为忘记释放资源而引起内存泄露。

使用裸指针，在资源不需要时，程序员要主动去销毁他们

```C++
// Need to create the object to achieve some goal
MyObject* ptr = new MyObject(); 
ptr->DoSomething(); // Use the object in some way
delete ptr; // Destroy the object. Done with it.
// Wait, what if DoSomething() raises an exception...?
```

指针指针则定义了何时去销毁对象的规则。你还是要去创建对象，但是你不用操心对象的销毁了。

```C++
SomeSmartPtr<MyObject> ptr(new MyObject());
ptr->DoSomething(); // Use the object in some way.

// Destruction of the object happens, depending 
// on the policy the smart pointer class uses.

// Destruction would happen even if DoSomething() 
// raises an exception
```

实践中最简单的规则是被管理对象的生命周期等同于智能指针对象的生命周期。比如 [boost::scoped_ptr](http://www.boost.org/doc/libs/release/libs/smart_ptr/scoped_ptr.htm) 或者 [std::unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr) 就实现了这个规则。

```C++
void f()
{
    {
       boost::scoped_ptr<MyObject> ptr(new MyObject());
       ptr->DoSomethingUseful();
    } // boost::scopted_ptr goes out of scope -- 
      // the MyObject is automatically destroyed.

    // ptr->Oops(); // Compile error: "ptr" not defined
                    // since it is no longer in scope.
}
```

注意 scoped_ptr 的对象是禁止拷贝的，这可以防止多次（错误的）释放同一个对象。如果需要，可以通过传递引用的方式把他们传入被调用的函数中。

基于作用域的智能指针在你需要把对象的生命周期绑定到一个代码块上或者另一个对象时（把对象嵌入到另一个对象中）很有用。对象在代码块执行完之前或绑定的对象被销毁之前会一直存在。

更复杂的智能指针采取引用计数的策略。实现这种策略的智能指针能够被复制，当对象的最后一个引用被销毁时才去执行对象的删除。[boost::shared_ptr](http://www.boost.org/doc/libs/release/libs/smart_ptr/shared_ptr.htm) 和 [std::shared_ptr](http://en.cppreference.com/w/cpp/memory/shared_ptr) 实现了这个策略。

```C++
void f()
{
    typedef std::shared_ptr<MyObject> MyObjectPtr; // nice short alias
    MyObjectPtr p1; // Empty

    {
        MyObjectPtr p2(new MyObject());
        // There is now one "reference" to the created object
        p1 = p2; // Copy the pointer.
        // There are now two references to the object.
    } // p2 is destroyed, leaving one reference to the object.
} // p1 is destroyed, leaving a reference count of zero. 
  // The object is deleted.
```

基于引用计数的智能指针在对象的生命周期并不是直接绑定到特定代码段或者特定对象时很有用。

基于引用计数的智能指针也有相应的缺点，比如可能会创建出悬空的引用：

```C++
// Create the smart pointer on the heap
MyObjectPtr* pp = new MyObjectPtr(new MyObject())
// Hmm, we forgot to destroy the smart pointer,
// because of that, the object is never destroyed!
```

另一种可能是出现循环引用：

```C++
struct Owner {
   boost::shared_ptr<Owner> other;
};

boost::shared_ptr<Owner> p1 (new Owner());
boost::shared_ptr<Owner> p2 (new Owner());
p1->other = p2; // p1 references p2
p2->other = p1; // p2 references p1

// Oops, the reference count of of p1 and p2 never goes to zero!
// The objects are never destroyed!
```

To work around this problem, both Boost and C++11 have defined a weak_ptr to define a weak (uncounted) reference to a shared_ptr.

为了解决这个问题 Boost 和 C++11 都提供了 weak_ptr 用于创建 shared_ptr 的 weak （并不算有效计数） 引用。

### 更新

这个回答已经过时了， 从 C++11 开始 标准库提供了一整套的智能指针，所以你应该使用 [std::unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr), [std::shared_ptr](http://en.cppreference.com/w/cpp/memory/shared_ptr) 和 [std::weak_ptr](http://en.cppreference.com/w/cpp/memory/weak_ptr).

std::auto_ptr 它类似于基于作用域的智能指针，但是有很多问题。**在新标准中该指针已经被废弃了，在任何时候不要再使用它**！

```C++
std::auto_ptr<MyObject> p1 (new MyObject());
std::auto_ptr<MyObject> p2 = p1; // Copy and transfer ownership. 
                                 // p1 gets set to empty!
p2->DoSomething(); // Works.
p1->DoSomething(); // Oh oh. Hopefully raises some NULL pointer exception.
```