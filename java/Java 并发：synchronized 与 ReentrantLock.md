## Java 并发：synchronized 与 ReentrantLock

> 2019/12/21

#### synchronized 概述

synchronized 的使用：

- 修饰方法，锁的是当前方法的对象
- 修饰静态方法，锁的是当前类的 Class 对象
- 修饰代码块，锁的是括号中的对象

synchronized 在字节码层级的原理：

在修饰代码块的时候：

编译下面的代码：

```java
public class Test {

    private static final Object obj = new Object();

    public static void main(String[] args) {
        int num = 1;
        synchronized (obj) {
            num = num + 1;
        }
    }
}
```

main 方法的字节码 Code 属性：

```java
  Code:
      stack=2, locals=4, args_size=1
         0: iconst_1          
         1: istore_1          
         2: getstatic     #2                  
         5: dup
         6: astore_2
         7: monitorenter  // monitorenter
         8: iload_1
         9: iconst_1
        10: iadd
        11: istore_1
        12: aload_2
        13: monitorexit  // monitorexit
        14: goto          22
        17: astore_3
        18: aload_2
        19: monitorexit  // monitorexit
        20: aload_3
        21: athrow
        22: return
```

可以看到有一个 monitorenter，两个 monitorexit 指令

- monitorenter：synchronized 大括号开始执行 monitorenter 指令，开始锁定

- monitorexit：synchronized 大括号结束执行 monitorexit 指令，自动解锁

两个 monitorexit 一个是在大括号结束后释放锁，一个是在代码异常时释放锁。

在 synchronized 修饰方法的时候，在方法的 flags 属性上会添加一个 ACC_SYNCHRONIZED 标志，当虚拟机访问一个 ACC_SYNCHRONIZED 标志的方法时，会在方法开始、结束、异常退出的位置添加 monitorenter 和 monitorexit 指令。

根据虚拟机规范的要求，在执行 monitorenter 指令时，首先要尝试获取对象的锁，如果这个对象没被锁定，或者当前线程已经拥有了这个对象的锁，把锁的计数器加 1，相应的，在执行 monitorexit 指令时会将锁计数器减 1，当计数器为 0 时，锁就被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到对象锁被另外一个线程释放为止。

这个过程是通过一个线程指针和一个计数器来实现的，monitor 获取锁时，将线程指针指向当前线程，计数器值加 1，monitor 释放锁时，将线程指针也释放，计数器值减 1。

#### ReentrantLock

ReentrantLock 需要手动去 lock 和 unlock，这是因为在发生异常的时候 synchronized 会自动释放锁，而 ReentrantLock 不会，所以对于  unlock 操作，我们需要把它放在 finally 中。

```java
ReentrantLock lock = new ReentrantLock();
try {
  lock.lock()
} finally {
  lock.unlock()
}
```

synchronized 和 ReentrantLock 默认都是非公平锁，但是 ReentrantLock 可以通过传入一个参数来实现公平锁，所谓公平锁就是让多个线程按照申请锁的顺序获取锁。

#### ReentrantReadWriteLock

读写锁，ReentrantReadWriteLock 最大的特点就是可以同时创建两个锁，一个读锁、一个写锁，在操作写锁的时候，读锁会被阻塞。

```java
ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
try {
  readWriteLock.readLock().lock();
  // 读数据
} finally {
  readWriteLock.readLock().unlock();
}

try {
  readWriteLock.writeLock().lock();
  // 写数据, 在这里, 如果有其他线程读数据的话, 会被阻塞
} finally {
  readWriteLock.writeLock().unlock();
}
```

#### synchronized 与 Lock 的区别

JDK1.5 之前，Java 通过 synchronized 关键字来实现锁的功能，synchronized 不需要我们手动去获取锁和释放锁，都是由 JVM 隐式帮我们实现的，在 JDK 1.5 之后，Java 增加了 Lock 来实现锁的功能，但是需要我们显示的去获取和释放锁。

JDK 1.5 时，Lock 是基于 Java 实现的，synchronized 是基于操作系统底层的互斥锁实现，在每次获取锁和释放锁都会带来操作系统用户态、内核态的切换，因此性能相对 Lock 差很多。

在 JDK 1.6 时，Java 对 synchronized 进行了优化，synchronized 有了一个锁升级的过程，性能提升了很多。

他们的主要区别如下：

- synchronized 异常会自动释放锁，Lock 不会，所以要在 finally 中使用

- synchronized 是非公平锁，Lock 可以实现公平也可以实现非公平

- Lock 提供了一些 API 可以获取锁的状态，synchronized 不行

  ```java
  boolean isLocked()  // 当前的锁是否处于 lock 状态
  Thread getOwner()   // 返回获取锁的线程
  ```

- 最重要的是，与 synchronized 搭配使用的 wait、notify 不能唤醒指定的线程，而 Lock 可以通过 Condition 做到这一点。
