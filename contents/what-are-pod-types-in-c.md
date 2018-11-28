# C++ 中的 POD 类型是什么？

## 回答

POD 是 Plain Old Data 的缩写，是指一个 没有构造函数，析构函数和虚函数的类 （不管是 使用 class 关键字还是 struct 关键字定义）。 [WIKI](http://en.wikipedia.org/wiki/Plain_Old_Data_Structures)这篇文章有更详细的说明

>A Plain Old Data Structure in C++ is an aggregate class that contains only PODS as members, has no user-defined destructor, no user-defined copy assignment operator, and no nonstatic members of pointer-to-member type.

>C++ 中的 POD 类型是只包含 POD 成员的聚合类，没有用于自定义的析构函数，拷贝构造函数和非静态的指向成员的指针成员。([pointer-to-member](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.cbclx01/cplr034.htm))。

更详细的内容可以参考这篇回答 [this answer for C++98/03](https://stackoverflow.com/a/4178176/734069), C++11 修改了 POD 的相关规定， 大大的放松了， 可以参考 [ necessitating a follow-up answer here](https://stackoverflow.com/a/7189821/734069)。