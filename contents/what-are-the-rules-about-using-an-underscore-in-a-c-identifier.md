# C++ 标识符中对下划线的使用有什么规定？

## 回答

规定（C++11 并没有变更）

- 任何范围内均保留使用，包括 [C++ 实现](https://stackoverflow.com/questions/4297933/c-implementation#4297974)的宏：
    - 以下划线开头后面紧跟着一个大写字母的标识符
    - 包含 相邻 下划线的标识符（或者 “double underscore”（是指仅仅由两个下划线命名的标识符么？））
- 在全局命名空间保留
    - 以下划线开头的标识符
- 同样所有 std 命名空间的标识符是保留的（你可以添加模板的特例化实现）

引用至 C++03 标准：
>17.4.3.1.2 Global names [lib.global.names]  
Certain sets of names and function signatures are always reserved to the implementation:  
Each name that contains a double underscore (__) or begins with an underscore followed by an uppercase letter (2.11) is reserved to the implementation for any use.  
Each name that begins with an underscore is reserved to the implementation for use as a name in the global namespace.    
Such names are also reserved in namespace ::std (17.4.3.1).

因为 C++ 标准是基于 C 标准的，下面是 C99的相关内容引用

>7.1.3 Reserved identifiers  
Each header declares or defines all identifiers listed in its associated subclause, and optionally declares or defines identifiers listed in its associated future library directions subclause and identifiers which are always reserved either for any use or for use as file scope identifiers.  
All identifiers that begin with an underscore and either an uppercase letter or another underscore are always reserved for any use.  
All identifiers that begin with an underscore are always reserved for use as identifiers with file scope in both the ordinary and tag name spaces.  
Each macro name in any of the following subclauses (including the future library directions) is reserved for use as specified if any of its associated headers is included; unless explicitly stated otherwise (see 7.1.4).  
All identifiers with external linkage in any of the following subclauses (including the future library directions) are always reserved for use as identifiers with external linkage.(The list of reserved identifiers with external linkage includes errno, math_errhandling, setjmp, and va_end.)   
Each identifier with file scope listed in any of the following subclauses (including the future library directions) is reserved for use as a macro name and as an identifier with file scope in the same name space if any of its associated headers is included.  
No other identifiers are reserved. If the program declares or defines an identifier in a context in which it is reserved (other than as allowed by 7.1.4), or defines a reserved identifier as a macro name, the behavior is undefined.  
If the program removes (with #undef) any macro definition of an identifier in the first group listed above, the behavior is undefined.  

同样存在其他的限制，比如 POSIX 标准保留了很多可能在普通代码中可能出现的标识符

- 以大写字母E开头后续紧跟数字或者大写字母:
    - 可能用作附加错误码的名字.
- 以 is 或 or 开头 后面紧跟小写字母
    - 可能用于附加字符测试或者转换函数
- 以 LC_ 开头 后面紧跟大写字母
    - 可能用于表示本地化的宏
- 所有以 f 或者 l 为后缀的数学函数名
    - 对应的接受参数是 float 或者 long double 的数学函数
- 以 SIG 开头后面紧跟大写字母
    - 附加的信号名
- 以 SIG_ 开头后面紧跟大写字母
    - 附加的信号名
- 以 str, mem, or wcs 开头紧跟小写字母或者
    - 附加的字符串或者数组函数
- 以 PRI or SCN 紧跟 小写字母或者 X
    - 附加的格式化类型宏
- 以 _t 结尾的标识符
    - 附加的类型名

现在在你的代码中使用这些标识符可能并没有问题，但是这引发了和未来版本标准库冲突的可能性。

---

就我本人来说，我从来不是用下划线开头的标识符。新增的个人守则：任何地方不要使用 double 下划线，这对我来说没问题，因为我很少使用下划线。

完成本文的研究之后，我以后不会再以 _t 来结尾我的标识符了，因为他们被 POSIX 标准保留了

保留所有 _t 结尾的标识符让我感到很奇怪。I think that is a POSIX standard (not sure yet) looking for clarification and official chapter and verse. This is from the GNU libtool manual, listing reserved names.

CesarB provided the following link to the POSIX 2004 reserved symbols and notes 'that many other reserved prefixes and suffixes ... can be found there'. The POSIX 2008 reserved symbols are defined here. The restrictions are somewhat more nuanced than those above.