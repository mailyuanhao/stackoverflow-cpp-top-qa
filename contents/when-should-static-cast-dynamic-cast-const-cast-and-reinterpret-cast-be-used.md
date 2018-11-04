# static_cast, dynamic_cast, const_cast and reinterpret_cast 分别在什么场景下使用？

下列转换的各自用途是什么？

- static_cast
- dynamic_cast
- const_cast
- reinterpret_cast
- C-style cast (type)value
- Function-style cast type(value)

如何决定在哪种场景下使用哪种转换方式？

## 回答

- static_cast 应该是优先尝试使用的转换。它做的事情类似隐私类型转换（比如 int 转换为 float， 指针转换为 void* )， 它同样可以用来调用显示转换方法 (explicit conversion functions) 或者隐式转换。在很多情况下明确声明 static_cast 并不是必须的。T(something) 在语法上等同于 (T)something， 这两种语法要避免使用。T(something, something_else) 这种是安全的，会明确的调用某个构造函数。

static cast 同样可以用于继承层级之间的转换。可以用于向下转换，只要不跨越 virtual 继承。 把一个对象向下转换为他不属于的类型时 sttic_cast 并不做检查， 这种行为是未定义的。

- const_cast 可以用于把一个 const 变量 转为 非 const 的或者反过来。包括 reinterpret_cast 在内的其他 C++ 转换符都不能用于去除 const 修饰。 It is important to note that modifying a formerly const value is only undefined if the original variable is const; if you use it to take the const off a reference to something that wasn't declared with const, it is safe. 对一个之前是 const 的值进行修改是未定义的行为仅仅当原始变量是 const 的时候。如果对一个指向 非 const 的变量的引用 去除 const 修饰，这样是安全的。（换句话说， 无论怎么转换，只要最终修改操作施加的变量不是 const 的。就是安全的）。it is safe. This can be useful when overloading member functions based on const, for instance. It can also be used to add const to an object, such as to call a member function overload.

const_cast 可以同样用于 volatile ，虽然这种用法不常见。

- dynamic_cast 基本上只能用于处理多态。可以把指向多态类型的指针或者引用转变为其他的 类 类型（多态类型是指至少 声明或者继承了一个虚函数）。 它不仅仅用于向下转换，还可以用于像同级的类型，甚至另一个继承链上转换。它会在要转换的对象内查抄目标类型对应的对象（RTTI），如果可以查找到则返回指向目标对象的指针或者引用，如果转换不成功， 指针会返回 nullptr， 转换引用会抛出 std::bad_cast 异常。

dynamic_cast 同样存在一些限制。当遇到没有使用 虚拟继承的 菱形继承时（一个子对象中会包含多个被菱形继承的父类型的对象实体，造成无法确定返回哪个对象。），他无法正常的工作。它也只能用于处理 public 继承， 遇到 protected 和 private 继承时也不能正常工作。不过这些情况很少遇到。

- reinterpret_cast 是最危险的一种转换，应该尽量少的使用。它直接把一种类型转换为另一种类型，比如把对指针类型进行转换，在 int 中存储一个指针，或者其它各种令人恶心的操作。它唯一能保证的是再转换为原始类型时，得到
- 和之前一样的值（前提是转换的中间类型长度不小于原始类型）。它主要用于一些特别奇怪的转换或者比特操作，比如把一个裸数据流转换为特定数据类型，或者把数据存储到对齐指针的低位中（storing data in the low bits of an aligned pointer）.

C-style cast and function-style cast are casts using (type)object or type(object), respectively. A C-style cast is defined as the first of the following which succeeds: C 风格转换被定义为下列转换中第一个能成功实施的转换（我理解就是依次尝试下面的转换，哪一种能成功，这次转换就使用那一种转换）：

- const_cast
- static_cast (though ignoring access restrictions)
- static_cast (see above), then const_cast
- reinterpret_cast
- reinterpret_cast, then const_cast

C 风格转换在一些情况下可以用于替换其他的转换， 但是很危险，因为它很可能最终实施了 reinterpret_cast 这种情况下使用明确的转换方式会更好。优先考虑使用明确的转换方式。

C 风格转换在进行 static_cast 时会忽略访问控制，所以它可以进行其它转换操作完成不了的转换。这也是我认为应该尽量避免使用 C 风格转换的一个原因。

## 译者注

这篇译文翻译的问题百出，很多地方不知道怎么表述，有个别地方未能理解原作者的具体意思。

下面是我个人的理解：

关于 dynamic_cast，做转换时，是利用运行时类型识别在对象内部查找转换目标类型对应的对象在原对象中的偏移（可正，可负），然后返回目标对象的地址，引用。（这个和编译器的具体实现有关，可以参考 C++ 对象模型的一些知识。）

```C++
class A {public: virtual ~A{}{}}；
class B: public A {}；
class C: public B {};
class X {};
class Y: public X{};
class Z: public Y{};
class I: public C, public Z{};
class J: public I{};
class K: public J{};

C* ptr_c = new J;
Z* ptr_z = dynamic_cast<Z*>(ptr_c);
X* ptr_x = dynamic_cast<X*>(ptr_c);
I* ptr_i = dynamic_cast<I*>(ptr_c);
K* ptr_k = dynamic_cast<K*>(ptr_c);//返回NULL
K* ptr_k = static_cast<K*>(ptr_c);//返回错误的值，因为static_cast不做检查。
```

注意上述 class A 的虚析构函数是必须的，如果没有该虚析构函数，则整个继承链没有虚函数， 不满足 dynamic_cast 必须是多态类型的需求。

```C++
class A {};
class B:public A{};
class C:public A{};
class D: public B, public C{};
class E: public D {};

D* ptr_d = new E();
A* a = dynamic_cast<A*>(ptr_d);//错误， 非虚拟菱形继承， E对象实体中包含两个不同的 A 类对象，无法确定返回哪一个。
```

```C
class A {int x;};
class B {int y;};

class C : A, B{int *z;};

C c;
B* b = static_cast<B*>(&c);//编译错误，私有继承，无法访问 C 对象中私有的 B 子对象。
B* b = (B*)&c;//尝试做static_cast但是忽略访问控制。可以正常编译，但是打破了访问控制！！！！
```
