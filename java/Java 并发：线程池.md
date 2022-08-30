### Java 并发：线程池

> 2019/6/5
>
> [可以看下这篇文章，比我写的流畅](https://github.com/Blankj/AndroidOfferKiller/blob/master/java/%E7%BA%BF%E7%A8%8B%E6%B1%A0.md)

#### 使用线程池的好处

1. 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗；
2. 提高响应速度：当任务到达时，任务可以不需要等到线程创建就能立即执行；
3. 对线程进行管理和控制：避免过多的线程占用大量资源从而出现阻塞系统或者 OOM 的现象，提高系统资源的使用效率；提供定时、周期、单线程、并发数控制等功能。

#### ThreadPoolExecutor 创建线程池

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
  if (corePoolSize < 0 ||
      maximumPoolSize <= 0 ||
      maximumPoolSize < corePoolSize ||
      keepAliveTime < 0)
    throw new IllegalArgumentException();
  if (workQueue == null || threadFactory == null || handler == null)
    throw new NullPointerException();
  this.corePoolSize = corePoolSize;
  this.maximumPoolSize = maximumPoolSize;
  this.workQueue = workQueue;
  this.keepAliveTime = unit.toNanos(keepAliveTime);
  this.threadFactory = threadFactory;
  this.handler = handler;
}
```

- corePoolSize：线程池的核心线程数，一般情况下不管有没有任务都会一直在线程池中一直存活，在 allowCoreThreadTimeOut (boolean value) 设置为 true 时，闲置的核心线程会存在超时机制，如果在指定时间没有新任务来时，核心线程也会被终止，而这个时间间隔由第 3 个属性 keepAliveTime 指定。

  当向线程池提交一个任务时，若线程池已创建的线程数小于 corePoolSize，即便此时存在空闲线程，也会通过创建一个新线程来执行该任务，直到已创建的线程数大于或等于 corePoolSize 时，才会根据是否存在空闲线程，来决定是否需要创建新的线程。

- maximumPoolSize：线程池所能容纳的最大线程数，当活动的线程数达到这个值后，后续的新任务将会被阻塞，maximumPoolSize 必须大于等于 corePoolSize。

- keepAliveTime：控制线程闲置时的超时时长，超过则终止该线程。一般情况下用于非核心线程，在  allowCoreThreadTimeOut(boolean value) 设置为 true 时，也作用于核心线程。

- unit：用于指定 keepAliveTime 参数的时间单位。

- workQueue：线程池的任务队列，通过线程池的 execute(Runnable command) 方法会将任务 Runnable 存储在队列中。

  - 如果运行的线程数少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。
  - 如果运行的线程数等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。
  - 如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 maximumPoolSize，在这种情况下，任务将被拒绝。

  woekQueue 介绍：https://add7.cc/java/blockingqueue-gai-shu

  由上面的规则可以知道，使用 LinkedBlockingQueue，maximumPoolSize 是没有意义的，因为 LinkedBlockingQueue 永远不会满，非核心线程永远不会被创建。

- threadFactory：线程工厂，它是一个接口，用来为线程池创建新线程的。

- handler：线程池的拒绝策略，当线程池达到最大任务数，且任务队列饱和，再次提交任务时，会执行该策略，默认为 AbortPolicy。

  - AbortPolicy：抛出一个 RejectedExecutionException。

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
      throw new RejectedExecutionException("Task " + r.toString() +
                                           " rejected from " +
                                           e.toString());
    }
    ```

  - CallerRunsPolicy：用调用线程本身来执行此次提交的任务。

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
      if (!e.isShutdown()) {
        r.run();
      }
    }
    ```

  - DiscardPolicy：do nothing，什么也不做，也就是将任务丢弃。

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
    ```

  - DiscardOldestPolicy：丢掉最早提交的任务（马上要被执行的任务），然后重新提交这个任务。

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
      if (!e.isShutdown()) {
        e.getQueue().poll();
        e.execute(r);
      }
    }
    ```

#### 线程池的关闭

线程池提供了两个方法用于线程池的关闭：

- shutdown()：不会立即的终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

#### Java 的四种线程池

Java 通过 Executors 提供四种线程池，分别为：

- newCachedThreadPool 创建一个可缓存线程池，当任务到达时如果有可用线程，将直接使用该线程，否则将创建新的线程，空闲线程将在 60s 后被回收，因此，长时间保持空闲的线程池不会浪费资源。

  ```java
  public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
  }
  ```

- newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，空闲线程不会被回收，超出的线程会在队列中等待。

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
  }
  ```

- newScheduledThreadPool 创建一个最大线程为 Integer.MAX_VALUE 的线程池，支持定时及周期性任务执行。

  ```java
  public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
  }
  ```

- newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，当多个任务提交到单线程线程池中，线程池将逐个去进行执行。

  ```java
  public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
      (new ThreadPoolExecutor(1, 1,
                              0L, TimeUnit.MILLISECONDS,
                              new LinkedBlockingQueue<Runnable>()));
  }
  ```

#### Thanks

https://www.trinea.cn/android/java-android-thread-pool/

https://blog.csdn.net/wtopps/article/details/80682267

https://www.jianshu.com/p/b8197dd2934c

https://leokongwq.github.io/2017/02/26/java-ThreadPoolExecutor-RejectedExecutionHandler.html