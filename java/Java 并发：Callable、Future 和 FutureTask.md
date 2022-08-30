### Java 并发：Callable、Future 和 FutureTask

> 2019/6/18
>
> https://www.cnblogs.com/dolphin0520/p/3949310.html

####Callable 和 Runnable

Runnable 接口中 run 方法的返回值是 void，所以在执行任务之后我们无法获取返回结果。

```java
public interface Runnable {
    public abstract void run();
}
```

而 Callable 就不一样了，它执行之后会返回传进去的泛型。

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

Callable 通常和 ExecutorService 一起使用：

```java
<T> Future<T> submit(Callable<T> task);
```

#### Future

Future 的作用是对 Callable 的执行进行取消、查询是否完成、获取结果。

JDK 文档中给出的伪代码

```java
// 搜索的服务
interface ArchiveSearcher { 
		String search(String target); 
}

class App {
    // 创建线程池
    ExecutorService executor = ...
    // 创建一个搜索服务
    ArchiveSearcher searcher = ...
        
    void showSearch(final String target) throws InterruptedException {
        // 使用线程池执行 Callable
        Future<String> future = executor.submit(new Callable<String>() {
            public String call() {
                // 子线程中去搜索
                return searcher.search(target);
            }});
        displayOtherThings(); // do other things while searching
        try {
            displayText(future.get()); // use future
        } catch (ExecutionException ex) { 
          	cleanup(); 
          	return; 
        }
    }
}}
```

Future 接口详解

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

- cancel(boolean mayInterruptIfRunning)：尝试取消任务，如果任务已经完成、已经取消成功、或者因为一些原因取消失败，返回 false；如果任务还未执行，返回 true，并且任务不再会执行；如果任务在进行中，则取决于 mayInterruptIfRunning，mayInterruptIfRunning 表示是否可以取消正在进行的任务。
- isCancelled：表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true；如果 cancel 返回 true 这个方法一定返回 true。
- isDone：用来表示任务是否已经完成，正常完成、异常、取消成功都返回 true；在 cancel 返回后，这个方法一定返回 true。
- get()：获取 Callable 的执行结果，如果没有执行完，阻塞。
- get(long timeout, TimeUnit unit)：获取 Callable 的执行结果，如果在指定时间内没有执行完，返回 null。

总结，Future 的功能如下：

- 能够取消任务，而且知道是否取消成功。
- 能判断任务是否完成。
- 能获取任务结果。

#### FutureTask

FutureTask 类实现了 RunnableFuture 接口；

而 RunnableFuture 继承了 Runnable 接口和 Future 接口，因此，RunnableFuture 既可以作为 Runnable 被线程执行，又可以作为 Future 获取 Callable 的返回值。

FutureTask 不仅 Future 接口唯一的实现类，而且是一个 Runnable 接口。

FutureTask 也可以用来启动线程：

```java
// 这个本质也是使用 Runnable 接口
// FutureTask 实现了 Runnable 接口，会在 run 方法中调用 call 方法
new Thread(new FutureTask<String>(new Callable<String>() {
  @Override
  public String call() throws Exception {
    return null;
  }
}));
```





