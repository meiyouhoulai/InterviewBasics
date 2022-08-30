## Java 并发：AQS 与 ReentrantLock

#### 概述

AQS 全称为  Abstract Queued Synchronizer，一般翻译为同步器，是一套实现多线程同步的框架。

其中，ReentrantLock 内部就是通过 AQS 来实现的

ReentrantLock 通过一个内部类 Sync 来实现 AQS，Sync 分为 FairSync 和 NonfairSync，分别对应公平锁和非公平锁。

非公平锁的 lock 方法如下：

```java
final void lock() {
  // 1、通过 CAS 的方式设置同步状态的值, CAS 成功, 表示当前线程获取锁, 将线程设置为独占线程
  if (compareAndSetState(0, 1))
    setExclusiveOwnerThread(Thread.currentThread());
  else
    // 2、CAS 失败, 将线程加入等待队列
    acquire(1);
}

public final void acquire(int arg) {
  if (!tryAcquire(arg) &&
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}

protected final boolean tryAcquire(int acquires) {
  return nonfairTryAcquire(acquires);
}
```

