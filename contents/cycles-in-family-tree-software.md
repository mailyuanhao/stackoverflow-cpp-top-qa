# 族谱软件中的循环

## 问题

我开发了一个建立族谱的软件（使用 C++ 和 QT),最近遇到一个客户反馈 bug ，他说他和自己的女儿有两个孩子。

bug 的原因是代码中有 assert 确保 X 不能同时是 Y 的父亲和 祖父。请问怎么在不去除数据 assert 的前提下解决这个问题？

>这个问题已经锁定了,之所以还存在是因为历史原因。请不要以他为证据来证明你也可以提类似的问题。

翻译这个问题主要是下面有几个comment 太有意思了。

- 听起来，你就不应该把软件卖给这种有棘手问题的家庭。谁会和自己的女儿生两个孩子？我希望你说的是他的儿媳妇！
- 你在编写软件时，脑海中应该重复 [Ray Stevens](http://www.youtube.com/watch?v=eYlJH81dSiw) 的歌曲。
- 你应该扪心自问，你是否愿意和这个人做生意？另一个方案是对这个人提起刑事诉讼，乱伦在大多数国家是违法的。另外你的软件功能本身是有问题的，族谱是可以（合法）存在环的， 在大多数（全部？）西方国家，表亲是可以结婚的。

## 回答

不翻译了。这个题目本身意义不大。