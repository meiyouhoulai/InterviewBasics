## Java 的异常体系：Exception 和 Error

> 2019/2/22
>
> 1. try 的代码段大小是否会影响性能？
> 2. 什么是不进行栈快照的 Exceptiop，如何创建？

#### Throwable

Java 中的 Exception 和 Error 都继承了 Throwable，Java 中只有 Throwable 类型的实例才可以被抛出（throw）或者捕获（catch）。

#### Exception

The class `Exception` and its subclasses are a form of `Throwable` that indicates conditions that a reasonable application might want to catch.

Exception 指程序正常运行过程中，可以预料的意外情况，这种意外应该被捕获并进行相应的处理。

#### Error

Error 表示一种严重的意外情况，在正常情况下不应该出现，绝大多数 Error 都会导致程序处于非正常、不可恢复的状态。

they can occur at many points in the program and recovery from them is difficult or impossible。

因为 Error 是一种不应该出现的意外，又难以恢复，所以不便于也不需要进行捕获；

常见的 Error：OutOfMemoryError、StackOverflowError

#### RuntimeException

异常又分为检查性异常和非检查性异常，非检查性异常就是所谓的运行时异常，即 RuntimeException。

Java 语言中很多操作都可能出现 RuntimeException，RuntimeException 异常非常多，而 Java 在编译期又没办法确定会出现 RuntimeException，所以对于 RuntimeException 通常是在编码的时候避免，不强制要求在代码中进行显示的捕获处理。

常见的 RuntimeException：NullPointerException、ArrayIndexOutOfBoundsException、ClassCastException

#### Checked Exception

检查性异常，Exception 中除了 RunTimeException 都是 Checked Exceptions，如果一个方法或者构造方法可能抛出检查性异常，必须使用 throws 声明。

例如 InputStream 中的 read 方法：

```java
public abstract int read() throws IOException;
```

检查性异常在代码中必须显示地进行捕获处理，这是编译期检查的一部分，Java 这样做是为了减少代码中没有正确处理的异常数目。

#### Java 的异常体系

Throwable

- Exception
  - RunTimeException
  - Checked Exception
- Error

#### 关于异常需要注意的地方

1. 不要捕获 Exception，而是捕获特定异常，针对不同异常不同处理。
2. 不要生吞异常，捕获的异常一定要进行处理，尽量不要 ignored；可以使用自定义异常，保留原有的 cause 信息，不同的异常使用不同的 code 标记，然后根据 code 和具体的业务逻辑需要去处理异常。
3. 自定义异常的使用注意信息安全问题，比如 java.net.ConnectException，出错信息是类似 “ Connection refused (Connection refused)"，而不会出现具体的 IP、端口号等。类似的，像用户数据这种私密性的东西不要记录到我们的异常日志中。
4. 极客时间版权所有: https://time.geekbang.org/column/article/6849
5. 使用 `e.printStackTrace()` 记录异常日志不是一个好的办法，我们需要明确日志输出的位置。
6. 不要使用异常控制流程，相比 if else 效率差很多。

#### 关于异常的性能问题

1. try-catch 代码段会产生额外的性能开销，建议仅捕获有必要的代码段，尽量不要一个大的 try 包住整段代码。

   > 关于这一点有疑问，印象中 try 的代码段大小只是一个标记，不会影响性能。

2. Java 每实例化一个 Exception，都会对当时的栈进进行快照，这是一个相对比较重的操作。如果发生的非常频繁，这个开销可就不能被忽略了。

对于一些追求极致性能的底层库，有种方式是创建不进行栈快照的 Exception，其实这是一种不理智的行为，因为我们无法确定处理异常是是否需要堆栈信息，如果需要，这会大大增长处理异常的难度。

> 什么是不进行栈快照的 Exceptiop，怎么创建？

性能优化时，对发生最频繁的异常进行优化也是一种思路。

#### Thanks

https://docs.oracle.com/javase/9/docs/api/java/lang/Throwable.html

https://notendur.hi.is/snorri/SDK-docs/lang/lang073.htm

