## Java 并发：CAS

CAS 是 CompareAndSwap 的缩写，比较并替换。CAS 需要有 3 个操作数：内存地址 V，旧的预期值 A，即将要更新的目标值 B。

CAS 指令执行时，会比较内存地址 V 的值与预期值 A 是否相等，如果相等将内存地址 V 的值修改为 B，否则就什么都不做。

**整个比较并替换的操作是一个原子操作。**

CAS 最常见的使用是在 AtomicInteger 等原子类中，我们看下它的自增代码：

```java
public final int getAndIncrement() {
  return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

最终执行到了 Unsafe 类中的 getAndAddInt 方法

```java
public final int getAndAddInt(Object o, long offset, int delta) {
  int v;
  do {
    // 通过 o 和 offset 来确定内存地址
    // 从内存中获取旧值
    v = getIntVolatile(o, offset);
    // 使用新值比较并替换旧值成功后, 退出, 否则一直循环
  } while (!compareAndSwapInt(o, offset, v, v + delta));
  return v;
}
```

我们可以看到看到其中的关键是 compareAndSwapInt，这个方法是一个 native 方法，最终会调用 `Atomic::cmpxchg` 方法。

`Atomic::cmpxchg` 方法会判断当前的 CPU 是不是多核，如果是，在执行指令的时候会添加一个 lock 前缀，这样 CPU 在执行该指令的时候通过会总线锁定或者缓存锁定的方式保证原子性。

CAS 的有点：

无锁操作，不会有加锁、解锁的性能消耗；同是不会因为获取不到锁进入阻塞状态，以及触发线程切换。

CAS 的缺点：

1. 如果 CAS 一直失败，会导致循环时间很大，开销大
2. 只能保证一个共享变量的原子性，对操作多个共享变量无能为力
3. ABA 问题

ABA 问题：

ABA 问题就是内存中的值从 A 修改成了 B 又修改成了 A，此时 CAS 会认为这个值没有修改过。也就是说，CAS 无法判断这个值是否被更新过。

ABA 问题解决也很简单，就是添加一个版本号，在每次进行 CAS 操作的时候，我们可以手动去更新这个版本号。

Java 中提供了 AtomicStampedReference 和 AtomicMarkableReference 来解决 ABA 问题。

AtomicStampedReference 与 AtomicMarkableReference 区别也很简单，AtomicStampedReference 的版本好是一个 int 类型的变量 stamp，AtomicMarkableReference 将版本号简化成了一个 boolean 类型的 mark。

```java
AtomicStampedReference<Integer> atomicStampedRef = new AtomicStampedReference<>(100, 1);
boolean update = atomicStampedRef.compareAndSet(100, 101, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
System.out.println(update); // true
System.out.println(atomicStampedRef.getReference()); // 101
System.out.println(atomicStampedRef.getStamp()); // 2
```

