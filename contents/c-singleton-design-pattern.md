# C++ 单例设计模式

## 回答

2008 年我提供了一个 C++98 版本的一个 延迟计算，确保析构，非线程安全版本的单例模板。（ lazy-evaluated, guaranteed-destruction, not-technically-thread-safe） [Can any one provide me a sample of Singleton in c++?](https://stackoverflow.com/questions/270947/can-any-one-provide-me-a-sample-of-singleton-in-c/271104#271104)

下面是基于 C++ 11 的升级版实现，该版本的单例模板是延迟计算，正确析构而且[线程安全](https://stackoverflow.com/a/449823/52074)

```C++
class S
{
    public:
        static S& getInstance()
        {
            static S    instance; // Guaranteed to be destroyed.
                                  // Instantiated on first use.
            return instance;
        }
    private:
        S() {}                    // Constructor? (the {} brackets) are needed here.

        // C++ 03
        // ========
        // Don't forget to declare these two. You want to make sure they
        // are unacceptable otherwise you may accidentally get copies of
        // your singleton appearing.
        S(S const&);              // Don't Implement
        void operator=(S const&); // Don't implement

        // C++ 11
        // =======
        // We can use the better technique of deleting the methods
        // we don't want.
    public:
        S(S const&)               = delete;
        void operator=(S const&)  = delete;

        // Note: Scott Meyers mentions in his Effective Modern
        //       C++ book, that deleted functions should generally
        //       be public as it results in better error messages
        //       due to the compilers behavior to check accessibility
        //       before deleted status
};
```

参见下面的文章决定什么时候使用单例（不经常用）  
[Singleton: How should it be used](https://stackoverflow.com/questions/86582/singleton-how-should-it-be-used)

下面两篇文章是关于初始化顺序以及应对方法的  
[Static variables initialisation order](https://stackoverflow.com/questions/211237/c-static-variables-initialisation-order/211307#211307)  
[Finding C++ static initialization order problems](https://stackoverflow.com/questions/335369/finding-c-static-initialization-order-problems/335746#335746)

参考下面讲解生命周期的文章  
[What is the lifetime of a static variable in a C++ function?](https://stackoverflow.com/questions/246564/what-is-the-lifetime-of-a-static-variable-in-a-c-function)

下面的文章是讨论单例和线程实现相关的  
[Singleton instance declared as static variable of GetInstance method, is it thread-safe?](https://stackoverflow.com/questions/449436/singleton-instance-declared-as-static-variable-of-getinstance-method/449823#449823)

下面的文章解释为什么 锁的双重检测 在 C++ 里面行不通  
[What are all the common undefined behaviours that a C++ programmer should know about?](https://stackoverflow.com/questions/367633/what-are-all-the-common-undefined-behaviour-that-c-programmer-should-know-about/367690#367690)  
[Dr Dobbs: C++ and The Perils of Double-Checked Locking: Part I](http://www.drdobbs.com/cpp/c-and-the-perils-of-double-checked-locki/184405726)