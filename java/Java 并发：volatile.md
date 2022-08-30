### Java 并发：volatile

> 2019/6/14

#### 双检锁单例

```java
class Single {
  private static volatile Single instance = null;
  private Single() {}
  public static Single getInstance() {
    // 如果实例不存在，获取锁
    if(instance == null) { 
      synchronized(Single.class) {
        // 可能获取锁之前实例不存在，获取锁之后实例被其他线程创建，再判断一次
        if(instance == null) {
          instance = new Single();
        }
      }
    }
    // 返回该实例
    return instance;
  }
}
```

可以看到使用了 volatile 来修饰单例变量，原因在下面这行代码：

```java
instance = new Single();
```

看上去是一行代码，实际这并不是一个原子操作：

1. 给 Single 分配内存
2. 调用 Single 构造函数，初始化成员变量
3. 将 instance 指向分配的内存空间

#### volatile 解释

JDK 文档中是这么描述 volatile：

A field may be declared volatile, in which case the Java Memory Model ensures that all threads see a consistent value for the variable.

一个字段如果被声明为 volatile 类型，Java 内存模型将确保所有线程看到的值是一致的。

在 Java 内存模型中，线程可以把变量保存在本地内存中（如寄存器），而不是直接在主存中进行读写，这样就可能出现一个线程在主存中修改了一个变量的值，而另一个线程还在使用它寄存器中的变量值的拷贝。从而出现数据不一致的问题。

volatile 是如何解决这个问题的呢？通过内存屏障

内存屏障是一个 CPU 指令，它有两个作用：

- 确保一些特定操作的执行顺序。

  编译器和 CPU 可以在保证最后输出结果一致的情况下对指令进行重排序，使性能得到优化，插入一个内存屏障，相当于告诉CPU和编译器先于这个命令的必须先执行，后于这个命令的必须后执行，禁止这种重排序。

- 影响一些数据的可见性，强制更新不同 CPU 的缓存。

在我们是要 volatile 关键字时，Java 会在写操作后加一个写内存屏障的指令，在读操作前加入一个读内存屏障。这意味着：

- 写入数据时，JVM 会把线程本地内存中的数据刷新到主存
- 在读取数据时，JVM 会把该线程本地内存的数据置为无效，从主内存中读取数据。

#### volatile 为什么不能保证原子性

volatile 仅仅在读取写入的时候，保证了线程之间的可见性，但是在读取写入之前的操作时不安全的。

例如：Integer i;  i ++

一个自增操作，分为下面几步：

1. 读取变量的值保存到线程本地内存
2. 增加变量的值
3. 将变量的值写回

在使用了 volatile 后，仅仅能保证最后一步让其他线程可见，但是前面的步骤仍然是不安全的，可能中间有其他线程进行修改。

#### Thanks

https://www.infoq.cn/article/double-checked-locking-with-delay-initialization/
