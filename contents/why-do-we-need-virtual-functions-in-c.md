# C++ 为什么需要虚函数？

## 回答

下面是我关于为什么需要虚函数（而不是什么是虚函数）的理解

看一个场景，你有如下两个类

```C++
class Animal
{
    public:
        void eat() { std::cout << "I'm eating generic food."; }
};

class Cat : public Animal
{
    public:
        void eat() { std::cout << "I'm eating a rat."; }
};
```

main 函数如下

```C++
Animal *animal = new Animal;
Cat *cat = new Cat;

animal->eat(); // Outputs: "I'm eating generic food."
cat->eat();    // Outputs: "I'm eating a rat."
````

到现在为止一切还挺好是不是？ 动物吃 generic 食物， 猫吃老鼠，也不需要 虚函数。

让我们做一点小修改，eat() 函数由一个中间函数去调用

```C++
// This can go at the top of the main.cpp file
void func(Animal *xyz) { xyz->eat(); }
```

现在我们的 main 函数改为如下形式

```C++
Animal *animal = new Animal;
Cat *cat = new Cat;

func(animal); // Outputs: "I'm eating generic food."
func(cat);    // Outputs: "I'm eating generic food."
```

我们把 Cat 传入到了 func() 函数，但是他不再吃老鼠。我们应该重载 func() 函数让它接受一个 Cat 指针么？ 如果你需要从 Animal 继承出多个 animal 子类，那么每个子类都需要一个对象的 func() 函数。

解决方案是把 Animal 类中的  eat 函数 声明成为虚函数。

```C++
class Animal
{
    public:
        virtual void eat() { std::cout << "I'm eating generic food."; }
};

class Cat : public Animal
{
    public:
        void eat() { std::cout << "I'm eating a rat."; }
};
```

main 函数：

```C++
func(animal); // Outputs: "I'm eating generic food."
func(cat);    // Outputs: "I'm eating a rat."
```

解决！
