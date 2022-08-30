## Java 中 final、finally、 finalize 有什么不同？

> 2019/2/26

#### final

- final 修饰的类不能被继承
- final 修饰的方法不能重写

- final 修饰的变量不能被修改

final 的作用：

使用 final 修饰类和方法，明确告诉别人，一些类或者方法是不允许修改的，避免 API 使用者修改一些底层基础功能。

使用 final 修饰变量，可以避免因为意外的赋值出现的编程错误；同时 final 也可以用来保护只读数据，尤其是在并发中，因为明确的不能再次给 final 变量赋值，有利于减少一些额外的同步开销，因此 final 变量在第一次赋值后是线程安全的。

#### finally

finally 是 Java 保证代码一定要执行的一种机制，通常我们会使用 try-catch-finally 或者 try-finally 在 finally 代码块中进行一些关闭资源的操作。

#### finalize

finalize 是 Object 的一个方法，它设计的目的是保证对象在垃圾收集前完成特定资源的回收，finalize 机制现在已经不推荐使用，在 JDK 9 开始标记为 Deprecated。

带有非空的 finalize() 方法的类会被 JVM 特殊处理：当 GC 发现这样的类的一个实例已经不再被任何活的强引用所引用时，就会把它放入 finalizer queue，排队调用其 finalize() 方法。而当这样的一个实例的 fianlize() 方法被调用（并且该实例没有被复活）之后，下一次 GC 就可以把它看作普通对象来清理掉了。

所以一个带 finalize() 方法的类的实例，从已经失去所有强引用到真正被GC回收，通常要经历两次GC。

```java
public class Person {
    private String name;
    private String age;

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age='" + age + '\'' +
                '}';
    }

	// 实现类的 finalize 方法
    @Override protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("people finalize");
    }
}

class PhantomReferenceTest {

    public static void main(String[] args) {

        Person person = new Person();
        ReferenceQueue<Person> queue = new ReferenceQueue<>();
        // 创建虚引用，要求必须与一个引用队列关联
        PhantomReference<Person> pr = new PhantomReference<>(person, queue);
        person = null;

        System.gc();  // 第一次 GC，将实现 finalize 方法的虚引用加入 finalizer queue
        System.runFinalization(); // 强制调用已经失去引用的对象的 finalize 方法 
        System.gc();  // 第二次 GC，清除虚引用，这是虚引用会进入 ReferenceQueue

        // System.out.println(queue.poll());

        try {
            Reference ref = queue.remove(1000L);
            System.out.println(ref);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
// 运行结果：
// java.lang.ref.PhantomReference@2b193f2d
```

在 JDK 的文档中写道：最终确定 finalize 机制存在问题，finalize 使用不当可能会导致一些性能、死锁、挂起和资源泄露的问题；我们无法保证 finalize 在什么时候执行，执行是否符合预期，也无法取消 finalize 的执行。

因为我们无法保证 finalize 的执行过程，而 finalize 又被设计成在对象被垃圾收集前调用，于是 finalize 对象成了一个特殊对象，GC 不能像正常对象那样回收它，要对它进行一些额外的处理，可能这个对象需要经过多个垃圾回收周期才能被回收，这样的结果就是 finalize 阻碍了垃圾的快速回收，甚至导致垃圾大量堆积。



