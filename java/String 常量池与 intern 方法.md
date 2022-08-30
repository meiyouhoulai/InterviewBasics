## String 常量池与 intern 方法

> 2018/4/10

#### 结论

- new String 不会将创建的对象放入常量池。
- 直接赋值时，首先检查池中是否有值相同的字符串对象，如果有则直接从池中刚查找到的对象引用使用；如果没有则新建字符串对象，返回对象引用，并且将新创建的对象放入池中。
- 调用 intern 方法时，如果常量池中有等于此 String 对象的字符串（由 equals 方法确定），会直接返回池中的字符串。否则，会将此 String 对象添加到池中，并且返回此池中对象的引用。

```java
class StringEqualsTest {

    public static void main(String[] args) {
        // new String 并不会在常量池创建对象
        String str1 = new String("lxm");
        // 直接赋值使用的是常量池中的对象
        String str2 = "lxm";
        String str3 = "lxm";
        // intern 方法会将字符串对象放入常量池
        String str4 = str1.intern();
        // 下面的例子，在编译时会优化为 String str5 = "lxm";
        String str5 = "l" + "x" + "m";

        System.out.println(str1 == str2); // false
        System.out.println(str2 == str3); // true
        System.out.println(str2 == str4); // true
        System.out.println(str2 == str5); // true

    }
}
```

#### String s = new String("abc") 创建了几个对象？

2 个对象，第一个对象是 "abc" 字符串存储在常量池中，第二个对象在 JAVA 堆中的 String 对象。

### intern 方法

```java
public static void main(String[] args) {
    String s = new String("1");
    s.intern();
    String s2 = "1";
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4);
}

// 运行结果
java6: false false
java7: false true
```

#### Java6 解释

在 Java6 中，常量池是放在 Perm 区，new 出来的对象放在 Heap 区，而 Perm 去和 Heap 区是完全分开的，所以拿一个 JAVA Heap 区域的对象地址和字符串常量池的对象地址进行比较肯定是不相同的，即使调用 intern 方法也是没有任何关系的。

#### Java7 解释

Perm 区是一个静态的区域，主要存储一些加载类的信息、常量池、方法片段等内容，默认大小只有 4M，一旦大量使用 String.intern 方法会直接产生 java.lang.OutOfMemoryError: PermGen space 错误。

所以在 Java7 中，常量池转移到了 JAVA Heap 区，为什么要移动，太小是一个主要原因。

首先看上半部分

```java
String s = new String("1");   // 创建了两个对象，一个指向是常量池中的 “1” ，一个指向堆中的 “1”
s.intern();      // 将堆中的字符串 “1” 放入常量池，但是常量池已经有 “1” 了，什么也不执行
String s2 = "1"; // 将常量池中的 “1” 赋值给 s2
System.out.println(s == s2); // s 指向堆中的对象 “1”，s2 指向常量池中的对象 “1”，显然不相等
```

后半部分

```java
// 创建了 4 个对象，常量池中的 “1”，堆中的对象 “11”，两个 new 出来的匿名 String
String s3 = new String("1") + new String("1"); 
// 将堆中的 “11” 写入常量池，常量池中没有，现在常量池也在堆中，不会再次在常量池中创建一个 “11”
// 二是直接将 s3 的地址写入常量池
s3.intern();
// 将常量池中 “11” 对象的地址，也就是 s3 的地址赋值给 s4
String s4 = "11";
// s4 和 s3 是同一个地址，都指向堆中 "11" 的地址，相等
System.out.println(s3 == s4);
```

总结：关键点有两个，一是常量池的是在 Perm 中还是在 JAVA Heap 中，二是 intern 执行是否有效。

#### Thanks 

https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html