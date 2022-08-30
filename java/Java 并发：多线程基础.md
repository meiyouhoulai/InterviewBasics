## Java 并发：多线程基础

### 一. 概念   

#### 进程与线程的区别和联系

- 进程是操作系统分配资源的最小单位，线程是程序执行的最小单位
- 进程有自己独立的数据空间，而线程共享进程的数据空间，一个进程中至少要有一个线程

#### 多线程的好处与弊端
- 实现了多部分代码同时运行，提升了的效率
-  线程切换时需要成本的，如果线程太多，线程切换频繁，反而会浪费资源

#### 常用的线程调度算法

> https://blog.csdn.net/yangzhongblog/article/details/9881753

- 时间片轮转
- 先来先服务：有利于长作业，不利于短作业；有利于 CPU 繁忙作业，不利于 IO 繁忙作业
- 短作业优先：长作业的运行得不到保证
- 优先权调度：非抢占式优先，主要用于一些批处理中；抢占式优先，在一个作业运行过程中，出现了另一个优先权更高的作业，调用程序会放弃当前作业的执行，将 CPU 分配给另一个优先权高的作业，这种做法能更好的满足紧迫作业的要求。
- 高响应比优先调度：每轮计算一个作业的响应比，响应比 = ( 等待时间 + 要求服务时间 ) / 要求服务时间，该算法既照顾了短作业，又考虑了作业到达的先后次序，不会使长作业长期得不到服务，但是必须提前知道作业的长短。
- 多级反馈队列调度：设置多个就绪队列，第一个优先级最高、第二个次之、第三个最低，但是优先级越高的队列中，每个线程执行的时间片越短；当一个新的线程就绪后，先把它放在第一个队列的末尾，按照时间片轮转的原则等待，当轮到该线程执行时，如果它能在当前时间片执行完成，则正常退出系统，如果不能，则在时间片结束后进入第二个队列的末尾，依次循环；只有当第一个队列为空是，第二队列的线程才能得到运行，如果第二队列的线程运行时，又有新的线程进入到第一队列，则 CPU 会放弃当前正在执行的线程，并将其放入第二队列队列的末尾，去执行第一队列中的新的线程。前面的算法要么有一定的局限性，要么必须知道线程的运行时间长短，而多级反馈队列调度算法则不必事先知道各种线程所需的执行时间，而且还可以满足各种类型进线程的需要，因而它是目前被公认的一种较好的进程调度算法。

sleep(0)：触发操作系统立刻重新进行一次CPU竞争 ( 线程调度 )

#### JVM 的线程调度

线程调度一般分为两种

- 协同式调度：线程的执行时间由线程自己来控制，线程执行结束后，会主动通知操作系统将 CPU 切换到另一个线程上面。这种方式实现简单，线程都是串行的，没有线程并发、同步问题，也就发挥不了多核 CPU 的能力，而且如果一个线程出现了问题，可能会一直阻塞，导致其他线程也无法执行。
- 抢占式调度：每个线程有操作系统来分配执行时间和线程的切换。

JVM 使用的是：抢占式调度 + 时间片轮转 + 线程的优先级，优先级越高的线程，越容易被 CPU 选中。

#### JVM 启动时启动几条线程

- 执行 main 函数的线程，该线程的任务代码都定义在 main 函数中。  
- 负责垃圾回收的线程。  

## 二. 线程的创建方式  
#### 1. 继承Thread类  
1>. 定义一个类继承 Thread 类。   

2>. 覆盖 Thread 类中的 run 方法。   

3>. 直接创建 Thread 的子类对象创建线程。   

4>. 调用 start 方法开启线程并调用线程的任务 run 方法执行。    

#### 2. 实现Runnable接口          
1>. 定义类实现 Runnable 的接口。   

2>. 覆盖 Runnable 接口中的 run 方法，目的也是为了将线程要运行的代码存放在该 run 方法中。   

3>. 通过 Thread 类创建线程对象。   

4>. 将 Runnable 接口的子类对象作为实参传递给 Thread 类的构造方法。     

5>. 调用 Thread 类中 start 方法启动线程。start 方法会自动调用 Runnable 接口子类的 run方法。   

PS:

* 用继承方式有一个弊端，那就是如果该类本来就继承了其他父类，那么就无法通过 Thread 类来创建线程了。
* 实现 Runnable 接口避免了 Java 单继承的局限性，所以，创建线程的这方式较为常用。
* Thread 在创建的时候，该 Thread 就已经命名了。
* 可以通过 Thread 的 getName 方法获取线程的名称，名称格式：Thread-编号（从0开始）。

#### 3. 使用 FutureTask

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

## 三. 线程的同步    
#### 需求：模拟四个线程卖票
```java
public class TacketTest {
 	public static void main(String[] args) {
      	Tacket t = new Tacket();
      	new Thread(t).start();
      	new Thread(t).start();
      	new Thread(t).start();
      	new Thread(t).start();
	 }
}
class Tacket implements Runnable {
 	private int tacket = 100;
 	public void run() {
     	while(true) {
       		if(tacket > 0) {
            	try {
                 	Thread.sleep(10);
            	} catch (InterruptedException e) {
                 	e.printStackTrace();
            	}
            	System.out.println(tacket -- );
       		}
    	}
	}
}
```
运行结果：

```
4
3
2
1
0
-1
-2
```

票数出现了负数，这是因为 Thread-0 通过了 if 判断后，在执行到 “tacket --” 语句之前，tacket 此时仍等于 1，CPU 切换到Thread-1、Thread-2、Thread-3 之后，这些线程依然可以通过 if 判断，从而执行 “tacket --” 的操作，因而出现了0、-1、-2 的情况。

#### 线程安全问题产生的原因
当一个线程中有多条语句在操作共享数据时，一个线程对多条语句只执行了一部分，还没有执行完，另一个线程
参与进来执行，导致共享数据错误，产生线程安全问题。

#### 解决办法
将多条操作共享数据的线程代码封装起来，当有线程在执行这些代码的时候，其他线程不可以参与执行，必须要当前线程把这些代码都执行完毕后，其他线程才可以参与执行。

Java 提供了同步代码块解决这个问题。

#### 同步代码块的格式

    synchronized(对象){
       // 需要被同步的代码;
    }

对象如同锁，只有锁的线程可以执行同步代码块，没有锁的线程，即使获取了CPU的执行权也进不去。

#### 同步的好处
解决了线程的安全问题，避免共享数据出现错误。

#### 同步的弊端
当线程相当多时，因为每个线程都会去判断同步上的锁，这是很耗费资源的，无形中会降低程序的运行效率。

#### 同步的前提
1. 必须有两个或者两个以上的线程
2. 必须使用同一个锁

修改后的run：

```java
public void run() {
  	while(true) {
       synchronized(obj) {
            if(tacket > 0) {
                 try {
                      Thread.sleep(10);
                 } catch (InterruptedException e) {
                      e.printStackTrace();
                 }
                 System.out.println(tacket -- );
            }
       }   
  	}
}
```
 运行结果：

 ![](http://i.imgur.com/FvWMIID.png)

#### 如何判断哪些代码需要同步?
1. 明确哪些代码是多线程执行的代码（ run方法 ）
2. 明确共享数据
3. 找到操作共享数据的多线程代码

#### 同步的另一种形式：同步函数
格式：在函数返回值类型前加上 synchronized

现在把卖票的代码封装成一个同步函数：

```java
public class TacketTest {
 	public static void main(String[] args) {
      	Tacket2 t = new Tacket2();
      	new Thread(t).start();
      	new Thread(t).start();
      	new Thread(t).start();
      	new Thread(t).start();
 	}
}
class Tacket2 implements Runnable {
 	private int tacket = 100;
 	// 同步函数
 	public synchronized void  saleTacket() {
     	 if(tacket > 0) {
           	try {
                Thread.sleep(10);
           	} catch (InterruptedException e) {
                e.printStackTrace();
           	}
           	System.out.println(tacket -- );
      	}
 	}
 
 	public void run() {
      	while(true) {
           	saleTacket();
      	}
 	}
}
```
运行结果：

![](http://i.imgur.com/pjtGGYs.png)

#### 同步函数用的是哪一个锁呢?
函数需要被对象调用，函数都有一个所属对象的引用 this，this 就是同步函数的锁

静态同步函数的锁是类的字节码文件 .class

## 四. 多线程下的单例设计模式
#### 单态设计模式
采取一定的方法保证在整个的软件系统中， 对某个类只能 存在一个对象实例，并且该类只提供一个取得其对象实例的方法

#### “懒汉式” 与 “饿汉式”的区别，是在与建立单例对象的时间的不同:
* “饿汉式” 是在不管你用的用不上，一开始就建立这个单例对象。

* “懒汉式” 是在你真正用到的时候才去建这个单例对象（实例的延迟加载），在多线程访问时会出现线程安全问题。

	* 直接使用同步函数或者同步代码块虽然能解决线程安全问题，但会每次获取实例都要判断锁，会降低懒汉式的效率，

	* 采用双重判断加锁的方式解决，锁使用的是该类的字节码文件对象.class


#### 饿汉式：
```java
// 饿汉式不存在线程安全问题
class Single{
   	private static final Single instance = new Single();  
 	  // final变量不能被赋值，s 终身指向这个固定的 Single 实例
   	private Single(){}
   	public static Single getInstance(){
 		return instance;
  	}
}
```


#### 懒汉式：

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

## 五. 死锁

线程一拿到锁 A 等待锁 B，而此时线程二拿到锁 B 等待锁 A，两个线程只有获取另一个锁后
执行完相应代码才会放弃自己手中的锁，这样就造成了一种相互等待的情形，称之为死锁。

死锁最常见的情形就是同步嵌套

```java
public class DeadLock {
 	public static void main(String[] args) {
      	new Thread(new LockRunnable(true)).start();
      	new Thread(new LockRunnable(false)).start();
 	}
}
class Lock {  // 定义A锁和B锁
 	public static Object LOCK_A = new Object();
 	public static Object LOCK_B = new Object();
}
// 同步嵌套死锁
class LockRunnable implements Runnable {
 	public boolean flag ;
 	public LockRunnable(boolean flag) {
      	this.flag = flag;
 	}
 	public void run() {
        // 一个线程只执行if部分
      	if(flag) {  
           	while(true) {
               	 synchronized(Lock.LOCK_A) {  
                	   // 线程获取A锁后，只有再获取 B 锁才能继续执行，但可能另一个线程获取了 B 锁，等待
                     System.out.println(Thread.currentThread().getName() + " 获取A锁,等待B锁");
                     synchronized(Lock.LOCK_B) {
                         System.out.println(Thread.currentThread().getName() + " 获取B锁");
                     }
                }   
           }
      	} else {  // 另一个线程只执行else部分
           	while(true) {
                synchronized(Lock.LOCK_B) { 
                     // 线程获取B锁后，发现 A 锁被其他线程获取，等待
                     System.out.println(Thread.currentThread().getName() + " 获取B锁,等待A锁");
                     synchronized(Lock.LOCK_A) {
                          System.out.println(Thread.currentThread().getName() + " 获取A锁");
                     }
                }   
           }
      	}   
 	}
}
```
运行结果：

![](http://i.imgur.com/2fwJafZ.png)



## 六. 线程间通讯与生产者消费者
多个线程在处理统一资源，但是任务却不同，这时候就需要线程间通信。
#### 等待/唤醒机制涉及的方法：
1. wait()：让线程处于阻塞状态，该线程必须具备锁，wait 后线程放弃锁被存储到等待队列中。
2. notify()：唤醒在此对象监视器上等待的单个线程（任何一个都有可能），被唤醒的线程进入锁池，参与锁的竞争。
3. notifyAll()：唤醒在此对象监视器上等待的所有线程。

#### P.S.

* 这些方法都必须定义在同步中，因为这些方法都与锁有关。一个锁上的 wait 线程，只能被该锁的 notify 唤醒，等待和唤醒必须是同一把锁。
* 必须要明确到底操作的是哪个锁上的线程！
* wait 和 sleep 区别?
    * wait 可以指定时间也可以不指定，sleep 必须指定时间。
    * 在同步中时，对 CPU 的执行权和锁的处理不同。
       * wait：释放执行权，释放锁。
       * sleep：释放执行权，不释放锁。
       
#### 为什么操作线程的方法wait、notify、notifyAll 定义在了 object 类中

因为这些方法是监视器的方法，监视器其实就是锁。
锁可以是任意的对象，任意的对象调用的方式一定在 object 类中。

#### 生产者消费者代码
```java
public class ProducerConsumer {
    public static void main(String[] args) {
        Resource res = new Resource();
        new Thread(new Producer(res)).start();
        new Thread(new Producer(res)).start();
        new Thread(new Consumer(res)).start();
        new Thread(new Consumer(res)).start();
    }
}


class Resource {
    private String name;   // 商品的名称
    private int count = 1; // 编号
    private boolean flag = false;  // true消费 false生产


    synchronized void set(String name) {  // 生产方法
        while (flag) {
            // 需要消费, wait 生产进程 ，使用 while，随时判断
            try {
                this.wait();
            } catch (Exception e) {
            }
            // 生产进程会在这里阻塞, 消费进程开始执行
        }
        this.name = name + "--" + count++;
        System.out.println(Thread.currentThread().getName() + " 生产了：" + this.name);
        flag = true;
        notifyAll();  // 为了保证唤醒消费进程，唤醒所有进程
    }


    public synchronized void out() {  // 消费方法
        while (!flag) {
            try {
                this.wait();
            } catch (Exception e) {
            }
        }
        System.out.println(Thread.currentThread().getName() + " 消费了：" + this.name);
        flag = false;
        notifyAll();
    }
}


class Producer implements Runnable {
    private Resource res;


    Producer(Resource res) {
        this.res = res;
    }


    public void run() {
        while (true) {
            res.set("+商品+");
        }
    }
}


class Consumer implements Runnable {
    private Resource res;


    Consumer(Resource res) {
        this.res = res;
    }


    public void run() {
        while (true) {
            res.out();
        }
    }
}
```
运行结果：

![](http://i.imgur.com/VbVBsmP.png)

#### 需要注意的问题：
判断标记使用 while，让被唤醒的线程再一次判断标记。
使用 if 会出现数据错误（比如连续生产了两次），wait 后被 notify 会从 wait 的下一句继续执行，不再判断标记，

唤醒线程使用 notifyAll，
因为需要唤醒对方线程，只用 notifyAll，容易出现只唤醒本方线程的情况。

#### 问题总结：
* if 出现数据错误
* while + notify 全部 wait

#### 利用数组解决生产者消费者：
```java
public class ProducerConsumerBuf {

    public static void main(String[] args) {
        Res res = new Res();
        new Thread(new MyPro(res)).start();
        new Thread(new MyPro(res)).start();
        new Thread(new MyCon(res)).start();
        new Thread(new MyCon(res)).start();
    }
}


class Res {
    public static final int N = 10;  // 数组大小
    private int[] buffer = new int[N];
    private int read = 0;  // 读指针，指向消费的商品
    private int write = 0; // 写指针，指向生产的商品

    private int count = 1; // 商品编号


    public synchronized void put() {
        while ((write + 1) % N == read) {  // 数组满暂停生产
            try {
                this.wait();
            } catch (Exception e) {
            }
        }
        buffer[write++] = count;  // 生产商品
        System.out.println(Thread.currentThread().getName() + ":" + count + " <<<<<");
        count = (count + 1) % N;
        if (write == N) { // 构成一个循环队列
            write = 0;
        }
        notifyAll();
    }


    public synchronized void get() {
        while (read == write) {
            try {
                this.wait();
            } catch (Exception e) {
            }
        }
        int data = buffer[read++];
        System.out.println(Thread.currentThread().getName() + ":" + ">>>>> " + data);
        if (read == N) {
            read = 0;
        }
        notifyAll();
    }
}


class MyPro implements Runnable {
    private Res res;


    MyPro(Res res) {
        this.res = res;
    }


    public void run() {
        while (true) {
            res.put();
            try {
                Thread.sleep(500);
            } catch (Exception e) {
            }
        }
    }
}


class MyCon implements Runnable {
    private Res res;


    MyCon(Res res) {
        this.res = res;
    }


    public void run() {
        while (true) {
            res.get();
            try {
                Thread.sleep(500);
            } catch (Exception e) {
            }
        }
    }
}
```

#### JDK1.5升级的生产者消费者
JDK1.5 中提供了多线程升级解决方案

* 将同步 Synchronized 替换成 Lock 的 lock 和 unlock 操作。

* 将 Object 中的 wait、notify、notifyAll 替换成 Condition 对象的 await、signal、signalAll。

* Lock：

    ```java
    lock        // 获取锁
    unlock  	// 释放锁
    newCondition()  // 创建一个Condition对象
    ```

* Condition： 绑定到锁上的监视器，可以用来将线程分组	

  以前 notify 唤醒随机线程，notifyAll 唤醒全部线程，现在可以用 signal 唤醒特定的线程

  ```java
  await();      // 等待
  signal(); 	  // 唤醒对应 await 的线程 
  signalAll();  // 唤醒所有等待线程
  ```

代码：

```java
public class ProducerConsumer_JDK5 {
    public static void main(String[] args) {
        Resource_5 res = new Resource_5();
        new Thread(new Producer_5(res)).start();
        new Thread(new Producer_5(res)).start();
        new Thread(new Consumer_5(res)).start();
        new Thread(new Consumer_5(res)).start();
    }
}


class Resource_5 {
    private String name;   // 商品的名称
    private int count = 1; // 编号
    private boolean flag = false;  //  true 消费 false 生产

    private final Lock lock = new ReentrantLock();
    private final Condition condition_pro = lock.newCondition();
    private final Condition condition_con = lock.newCondition();


    public void set(String name) {  // 生产方法
        lock.lock();
        try {
            while (flag) {  // 需要消费, wait生产进程
                condition_pro.await();
                // 生产进程会在这里阻塞, 消费进程开始执行
            }
            this.name = name + "--" + count++;
            System.out.println(Thread.currentThread().getName() + " 生产了：" + this.name);
            flag = true;
            condition_con.signal();  // 唤醒消费进程
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }


    public void out() {  // 消费方法
        lock.lock();
        try {
            while (!flag) {
                condition_con.await();
            }
            System.out.println(Thread.currentThread().getName() + " 消费了：" + this.name);
            flag = false;
            condition_pro.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
}


class Producer_5 implements Runnable {
    private Resource_5 res;


    Producer_5(Resource_5 res) {
        this.res = res;
    }


    public void run() {
        while (true) {
            res.set("+商品+");
        }
    }
}


class Consumer_5 implements Runnable {
    private Resource_5 res;


    Consumer_5(Resource_5 res) {
        this.res = res;
    }


    public void run() {
        while (true) {
            res.out();
        }
    }
}
```

## 七. 线程控制
#### 1. 停止线程
stop 已经过时，如何停止线程呢?  run方法结束。

stop 方法会 unlocak 线程上所有的锁，这种非正常的 unlock 会引发线程的 ThreadDeath 这个错误，并且会造成锁的状态不一致，这样会造成很多不可预料的结果。

run方法中通常具有循环，只要控制住循环，既可以让 run 方法结束，从而结束线程。

代码：

```java
public class StopThread {
	public static void main(String[] args) {
		StopRun sr = new StopRun();
		new Thread(sr).start();
		try {
			Thread.sleep(5);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		sr.stop(); // 线程运行5毫秒后停止
	}
}
class StopRun implements Runnable {
	private boolean flag = true;
	private int count = 1;
	public void stop() {
    	// 写一个停止方法，通过控制循环的标记让线程停下来
		flag = false;
	}
	public void run() {
		while(flag) {
			System.out.println(Thread.currentThread().getName() + ":" + count ++);
		}	
	}	
}
```
运行结果：

![](http://i.imgur.com/IAVIoOv.png)

#### 如果线程处于 wait 状态时不会读取到标记，如何停止呢? 

使用 interrupt 方法

```java
public void interrupt()
// 将线程从 wait()、join()、sleep(long)中强制唤醒，并收到一个 InterruptedException
// 如果一个线程在正常运行的话，interrupt 并不会中断线程
```
代码：

```java
public class StopWait {
    public static void main(String[] args) {
        StopWaitRun s = new StopWaitRun();
        Thread t = new Thread(s);
        t.start();
        t.interrupt();
    }
}


class StopWaitRun implements Runnable {
    private boolean flag = true;



    public synchronized void run() {
        while (flag) {
            // 线程运行后打印两句话进入等待状态
            System.out.println(Thread.currentThread().getName() + " 正在运行");
            System.out.println(Thread.currentThread().getName() + " 进入等待状态");
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName() + " 被强制唤醒");
                flag = false;
                System.out.println(Thread.currentThread().getName() + " 已经结束");
            }
        }
    }
}
```

运行结果：

![](http://i.imgur.com/Jtu427G.png)

使用 interrupt 结束线程的思想就是在 catch (InterruptedException e) {} 中停止线程。

#### 2. 加入线程

```java
public final void join()  // 等待该线程终止。
```

* join 的作用是阻塞当前线程，知道调用 join 方法的线程结束后，当前线程重新进入可运行状态。
* 当 A 线程执行到了 B 线程的 join() 方法时，A 就会等待。等 B 线程都执行完，A 才会执行。
* 线程必须先 start，join 才有效。
* join 可以用来临时加入线程执行。

代码：

```java
public class Join_Thread {
    public static void main(String[] args) {
        Thread t = new Thread(new JoinRun());
        t.start();
        try {
            t.join(); // 等到线程 t 执行结束后，主线程才执行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        for (int i = 0; i < 50; i++) {
            System.out.println(Thread.currentThread().getName() + ":" + i);
        }
    }
}


class JoinRun implements Runnable {
    public void run() {
        for (int i = 0; i < 50; i++) {
            System.out.println(Thread.currentThread().getName() + ":" + i);
        }
    }
}
```

运行结果：

![](http://i.imgur.com/HsbBF5e.png)

在 t 线程执行完毕 main 线程才开始执行

#### 3. 礼让线程

```java
public static void yield()  // 暂停当前正在执行的线程对象，并执行其他线程。 
```

* 注意这是一个静态方法使用 Thread.yield() 调用。
* 正在执行的线程让出 CPU 供其他线程执行。
* 实际上，当某个线程调用了 yield 方法后，只有优先级与当前线程相同，或者优先级比当前线程高的处于可运行状态的线程才可以获得执行的机会( 一个CPU的情况下 )
* 线程优先级：
  * 1 - 10  默认 5
  * 1最小，10最大
* MIN_PRIORITY        1
* NORM_PRIORITY       5
* MAX_PRIORITY        10

---

```java
public final void setPriority(int newPriority)  // 设置线程的优先级
public final int getPriority() // 返回线程的优先级。
```

代码：

```java
public class Yield_Thread {
	public static void main(String[] args) {
		Thread tHigh = new Thread(new YieldRun());
		Thread tMid = new Thread(new YieldRun());
		Thread tLow = new Thread(new YieldRun());
	
		// 设置三个线程的名称
		tHigh.setName(">>>>>>>>>>");
		tMid.setName(">>>>>");
		tLow.setName(">");
		tHigh.setPriority(Thread.MAX_PRIORITY); // 优先级10
		tMid.setPriority(Thread.NORM_PRIORITY); // 优先级5
		tLow.setPriority(Thread.MIN_PRIORITY);  // 优先级1
		tLow.start();
		tMid.start();
		tHigh.start();
	}
}
class YieldRun implements Runnable {
	public void run() {
		for(int i=0;i<50;i++) {
			System.out.println(Thread.currentThread().getName()+":"+i);
			Thread.yield(); // 让出 CPU
		}		
	}
}
```
运行结果：

![](http://i.imgur.com/dVjpbPp.png)

![](http://i.imgur.com/8zSsyUE.png)

可以看到高优先级线程比低优先级线程更容易被CPU选中执行


#### 5. 守护线程
```java
public final void setDaemon(boolean on)
// 将该线程标记为守护线程或用户线程，当正在运行的线程都是守护线程时，Java 虚拟机退出。 
// 该方法必须在启动线程前调用。 
// 守护线程通常做一些事情来辅助其他线程，没有单独存在的必要。
```
代码：

```java
public class Daemon_Thread {
	public static void main(String[] args) {
		Thread t = new Thread(new DaemonRun());
		t.setName("守护线程");
		t.setDaemon(true);
		t.start();
        // 当主线程执行完 t.start(); 主线程结束，系统中只剩下守护线程 t，JVM退出，t 结束
	}
}
class DaemonRun implements Runnable {
	public void run() {
		for(int i=0;i<5000;i++) {
			System.out.println(Thread.currentThread().getName()+":"+i);
		}
	}
}
```
运行结果：

![](http://i.imgur.com/Sc4Tvm9.png)

run方法原本是打印到5000的，因为只剩下守护线程JVM退出只来得及打印到3000多






















