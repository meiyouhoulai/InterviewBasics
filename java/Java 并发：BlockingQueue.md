### Java 并发：BlockingQueue

> 2019/5/30

一个特殊的队列，在获取元素的时候，如果队列为空会等待直到队列中有元素；在添加元素是，如果队列满会等待直到队列有可用空间。

BlockQueue 都是线程安全的，都不允许存储 null，BlockingQueue 通常作为 workQueue 和线程池一起使用。

#### 返回结果

BlockingQueue 对队列的特殊操作 ( 向满队列添加元素，或者从空队列获取元素 ) 方法有四种形式：

- Throws exception，抛出异常
- Special value，返回特殊值：true、false、null
- Blocks，阻塞
- Times out，超时后返回特殊值

#### 队列操作

##### 入队

```java
boolean add(E e);
boolean offer(E e);
void put(E e) throws InterruptedException;
boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;
```

##### 出队

```java
boolean remove(Object o);
E poll(long timeout, TimeUnit unit) throws InterruptedException;
E take() throws InterruptedException;
E poll(long timeout, TimeUnit unit) throws InterruptedException;
```

##### 获取元素

```java
E element();
E peek();
```

#### ArrayBlockingQueue 和 LinkedBlockingQueue

##### 相同点：

都是 BlockingQueue 的子类，拥有 Blocking 和 Queue 的能力；

Blocking 指的是进行特殊操作时有四种方式的返回结果；

Queue 指的是 FIFO ( first-in-first-out ) 先进先出，新元素插入队尾，每次出队的时候从队头开始。

##### 不同点：

- 底层数据结构不同

  ArrayBlockingQueue 底层由数组实现， 一旦创建，容量便不能被改变，非常适合用来处理固定容量的生产者消费者问题；

  LinkedBlockingQueue 底层由链表实现，容量可以设置，默认 Integer 最大值 。

- 锁机制不同

  ArrayBlockingQueue 的入队和出队使用的是一把锁；

  LinkedBlockingQueue 中锁是分离的，入队使用的是 putLock，出队使用的是 takeLock。

- 性能不同

  ArrayBlockingQueue 在入队和出队的时候直接将对象在数组中进行添加或者删除，

  LinkedBlockingQueue 在入队和出队的时候，需要创建额外的 Node 对象进行插入或者移除，数据量大的时候会出现频繁的 GC 影响性能。

- 吞吐量 ( 单位时间内成功地传送数据的数量 ) 不同

  因为 BlockingQueue 是阻塞性的队列，在数量量大于 ArrayBlockingQueue 的容量时，LinkedBlockingQueue 的吞吐量更大一些；

  反之在数量量小于 ArrayBlockingQueue 的容量时，ArrayBlockingQueue 的吞吐量更大一些。

#### SynchronousQueue

同步阻塞队列，SynchronousQueue 是一个没有数据缓冲的 [BlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/BlockingQueue.html) ，生产者线程对其的插入操作 put 必须等待消费者的移除操作 take，反过来也一样，类似一手交钱一手交货。

SynchronousQueue 中不允许放入 null，SynchronousQueue 不能够迭代，因为它不存储数据。

SynchronousQueue 通常用于 newCachedThreadPool 这个线程池中。

#### PriorityBlockingQueue

优先级队列，不允许 null 元素，队列中的元素必须实现 Comparable，出队的时候按照 Comparable 的自然顺序从小到大以次取出。

#### DelayQueue

延迟队列，队列内元素必须实现 Delayed 接口，Delayed 接口继承了Comparable<Delayed\>。

DelayQueue 的内部实现是一个 PriorityQueue，不允许存放 null，DelayQueue 出队顺序由 PriorityQueue 决定，只有在 Delayed 的延迟期满时才能从中提取元素，否则将一直阻塞。

```java
class DelayAble implements Delayed {

  private long delay;
  private long start;
  int priority;


  DelayAble(long delay, int priority) {
    this.delay = delay;
    this.start = delay * 1000 + System.currentTimeMillis();
    this.priority = priority;

  }


  // 在 getDelay 的返回值 <= 0 是，才能从 DelayQueue 中获取元素，否则将阻塞
  // 注意 getDelay 方法一定要返回一个变化的值，如果返回一个大于 0 的常值将一直阻塞
  @Override
  public long getDelay(@NotNull TimeUnit unit) {
    return start - System.currentTimeMillis();
  }


  // compareTo 方法觉得了 DelayQueue 的出队顺序，和 PriorityQueue 一样按照元素的自然顺序出队
  @Override
  public int compareTo(@NotNull Delayed o) {
    DelayAble delayAble = (DelayAble) o;
    return priority - delayAble.priority;
  }
}
```

