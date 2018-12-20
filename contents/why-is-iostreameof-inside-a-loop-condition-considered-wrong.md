# 为什么在循环条件中使用 iostream::eof 被认为是错误的？

## 回答

因为 iostream::eof 仅仅在已经读取到 stream 的结尾后才会返回 true，它并不能标示：下一次从 stream 中读取将会到达结尾。

考虑下面代码（假设下次次读取将会到 strema  的结尾）

```C++
while(!inStream.eof()){
  int data;
  // yay, not end of stream yet, now read ...
  inStream >> data;
  // oh crap, now we read the end and *only* now the eof bit will be set (as well as the fail bit)
  // do stuff with (now uninitialized) data
}
```

和下面代码对比

```C++
int data;
while(inStream >> data){
  // when we land here, we can be sure that the read was successful.
  // if it wasn't, the returned stream from operator>> would be converted to false
  // and the loop wouldn't even be entered
  // do stuff with correctly initialized data (hopefully)
}
```

## 高赞评论

最主要的原因是：**我们还没读取到流结尾，并不表明下一次读取肯定能成功！！！**