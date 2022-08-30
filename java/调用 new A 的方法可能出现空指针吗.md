### 调用 new A() 的方法可能出现空指针吗？

> 2019/4/4
>
> https://www.zhihu.com/question/51244545/answer/126055789

可能。

```java
public class ReachabilityFenceTest {

    public static void main(String[] args) {
        for (int i = 0; i < 2000; i++) {
            try {
                new A().val();
            } catch (NullPointerException e) {
                System.out.println("NullPointerException at " + i);
                e.printStackTrace();
                return;
            }

        }
    }
}


class C {
    void val() {
    }
}


class B {
    private C c = new C();


    void cleanC() {
        this.c = null;
    }


    void val() {
        System.gc();
        System.runFinalization();
        this.c.val();
    }
}


class A {
    private B b = new B();


    @Override protected void finalize() throws Throwable {
        super.finalize();
        this.b.cleanC();
    }


    void val() {
        this.b.val();
    }
}
```

运行结果：

```java
NullPointerException at 387
java.lang.NullPointerException
	at reference.B.val(ReachabilityFenceTest.java:43)
	at reference.A.val(ReachabilityFenceTest.java:59)
	at reference.ReachabilityFenceTest.main(ReachabilityFenceTest.java:13)
```

分析：

注意这个空指针，不是 A 对象为 null ，而是 C 对象变成了 null。

new A() 是一个匿名对象，没有任何引用指向它，在调用它的 val 方法时候，B 的 val 方法会调用：

```java
System.gc(); // A 对象没有任何引用指向它，gc 会把 A 回收
System.runFinalization(); // 强制调用 A 的 finalize 方法
```

在 A 的 finalize 方法中，C 被置空，后续调用 C 的 val 方法时出现了空指针。