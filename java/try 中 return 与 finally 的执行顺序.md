## try 中 return 与 finally 的执行顺序

> 2019/2/27
>
> https://blog.csdn.net/ns_code/article/details/17485221

#### 结论

try 语句在返回前，将其他所有的操作执行完，保留好要返回的值，而后转入执行 finally 中的语句，而后分为以下三种情况：

- 如果 finally 中有 return 语句，则会将 try 中的 return 语句 ”覆盖“ 掉，直接执行 finally 中的 return 语句，得到返回值，这样便无法得到 try 之前保留好的返回值。
- 如果 finally 中没有 return 语句，也没有改变要返回值，则执行完 finally 中的语句后，会接着执行 try 中的return 语句，返回之前保留的值。
- 如果 finally 中没有 return 语句，但是改变了要返回的值，这里有点类似与引用传递和值传递的区别，分以下两种情况
  - 如果 return 的数据是基本数据类型或文本字符串，则在 finally 中对该基本数据的改变不起作用，try 中的 return 语句依然会返回进入 finally 块之前保留的值。
  - 如果 return 的数据是引用数据类型，而在 finally 中对该引用数据类型的属性值的改变起作用，try 中的return 语句返回的就是在 finally 中改变后的该属性的值。

#### 第三种情况的原因

> 以下为猜测，待验证

在 try 中将要 return 的值会保存在一个栈上，转而执行 finally 中的语句，执行完 finally 的语句，取出栈的保存的值返回。

基本数据类型或者 String 会直接保存在栈上，而引用数据类型仅仅将引用保存在栈上，真正的对象保存在堆上，因为保存在栈上的引用和 finally 中修改的引用是对应同一块堆内存，所以在在 finally 中修改引用类型 try 中的 return 的对象也会被修改。

