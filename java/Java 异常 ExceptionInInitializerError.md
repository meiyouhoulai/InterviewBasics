### Java 异常 ExceptionInInitializerError

> 2018/11/8
>
> https://docs.oracle.com/javase/9/docs/api/java/lang/ExceptionInInitializerError.html

An `ExceptionInInitializerError` is thrown to indicate that an exception occurred during evaluation of a static initializer or the initializer for a static variable.

ExceptionInInitializerError 在一般发生在静态变量初始化的时候。

静态变量初始化的顺序有先后之分，与代码中声明的顺序保持一致。如果初始化前一个静态变量的时候，引用的后面一个未初始化的静态变量就会抛出这个异常。

我出现异常的代码：

```kotlin
object FindDefault {

    @JvmStatic
    var wallet: Wallet = Wallet("")
    
    @JvmStatic
    var network: NetworkInfo = NetworkInfo.ETH
}

class Wallet(
    val address: String,
    val name: String = "",
    var networkInfo: NetworkInfo = FindDefault.network
)
```

分析：

在 FindDefault 中静态变量 wallet 初始化要早于 network，但是在 wallet 初始的时候会调用 FindDefault.network，此时 FindDefault.network 还没有初始化，抛出 ExceptionInInitializerError。

将 FindDefault 中 wallet 与 network 的声明顺序修改一下即可。

#### Thanks

https://blog.csdn.net/u010397369/article/details/16810863

