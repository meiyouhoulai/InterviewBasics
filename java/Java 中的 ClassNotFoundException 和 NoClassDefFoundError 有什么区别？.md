## Java 中的 ClassNotFoundException 和 NoClassDefFoundError 有什么区别？

> 2019/2/26

#### ClassNotFoundException

> https://docs.oracle.com/javase/9/docs/api/java/lang/ClassNotFoundException.html

ClassNotFoundException 继续 Excrption，是一种 Checked Exception，通常会在三个地方抛出：

- Class 的 forName 方法
- ClassLoader 的 findSystemClass 方法
- ClassLoader 的 loadClass 方法

ClassNotFoundException 表示我们的应用程序通过一个类的全类名去加载它，但是在 classpath 路径下却找不到这个类。

#### NoClassDefFoundError

> https://docs.oracle.com/javase/9/docs/api/java/lang/NoClassDefFoundError.html

NoClassDefFoundError 继承 Error，表示该类在编译阶段还可以找到，但是在运行的时候找不到了。

#### 总结

1、ClassNotFoundException 是一种 Checked Exception，NoClassDefFoundError 是一种 Error。    

2、ClassNotFoundException 是因为我们通过全类名加载的类不在 classpath 路径下，NoClassDefFoundError 是因为这个类在编译时期存在，在运行时期却不可用了。    

3、ClassNotFoundException 由程序代码抛出，NoClassDefFoundError 在 Java 运行时由 JVM 抛出。

