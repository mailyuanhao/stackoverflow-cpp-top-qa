# C++ 中的仿函数是什么？有什么作用？

## 回答

仿函数差不多就是一个定义了 operator() 的类，允许你创建看起来像 ”函数“ 的对象。

```C++
// this is a functor
struct add_x {
  add_x(int x) : x(x) {}
  int operator()(int y) const { return x + y; }

private:
  int x;
};

// Now you can use it like this:
add_x add42(42); // create an instance of the functor class
int i = add42(8); // and "call" it
assert(i == 50); // and it added 42 to its argument

std::vector<int> in; // assume this contains a bunch of values)
std::vector<int> out(in.size());
// Pass a functor to std::transform, which calls the functor on every element 
// in the input sequence, and stores the result to the output sequence
std::transform(in.begin(), in.end(), out.begin(), add_x(1)); 
assert(out[i] == in[i] + 1); // for all i
```

仿函数在多个方面有自己的优势，其中之一是和普通函数相比仿函数有状态。上面的例子创建了一个仿函数，功能是在你传入的任意值上面加 42 。 而且 42 并不是硬编码的，而是我们在创建仿函数对象时传入给构造函数的参数。我可以很轻易的创建另一个加 27 的仿函数，仅仅需要在创建时传入27 。 这使得仿函数特别容易定制。

就像例子中最后一行展示的，你经常要把仿函数当做参数传给别的函数，比如 std::transform 或者其他的标准哭库中的函数。你也可以传入普通的函数指针，不过我上面说过了，仿函数更加容易定制化，所以使用起来更加灵活（如果传入普通函数指针，那么我需要写一个专用的函数用来加 1 ， 而仿函数更加通用化，你初始化多少就加上多少）而且使用仿函数也可能拥有更高的性能。 在上面的例子中编译器知道 std::tranform 就是要调用 add_x::operator(), 这意味着它可以内联这个函数调用，使得性能和我们手动在 vector 的每个成员上调用函数的性能一致。

如果我传递的是一个普通的函数指针，除非编译器做非常复杂的全局优化，否则它无法得知具体要被调用的函数。所以只能在运行期进行解引用然后进行调用操作。
