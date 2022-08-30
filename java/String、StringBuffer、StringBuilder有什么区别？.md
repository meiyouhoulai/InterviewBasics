## String、StringBuffer、StringBuilder有什么区别？

> 2019/4/8

#### String

String 类提供了构造和管理字符串的基本逻辑，它是一个 final 类，不可继承，同时 String 类中负责存储字符串的的 `char value[]` 也被声明成 final 类型，因此，任何拼接、裁剪字符串的操作都会产生新的 String 对象，进而对应有的性能产生影响。

#### StringBuffer

相比 String，StringBuffer 解决了字符串拼接、剪裁都会产生新的字符串的问题，它提供了 append、delete 等方法来操作字符串，StringBuffer 对字符串的操作的方法都使用了 synchronized 来修饰以保证线程安全，但是也带来了额外的性能开销。

#### StringBuilder

StringBuilder 在使用上和 StringBuffer 没有区别，但是它去掉了线程安全的部分，有效减小了开销，是绝大部分情况下操作字符串的首选。

#### AbstractStringBuilder

AbstractStringBuilder 是 StringBuffer 和 StringBuilder 的父类，二者操作字符串的方法都是调用的这个类的方法，不同的是 StringBuffer 在方法上使用了 synchronized 修饰。

#### 字符串常量池

由于 String 在 Java 中使用过于频繁，Java 为了避免在一个系统中产生大量的 String 对象，引入了字符串常量池。

其运行机制是：创建一个字符串时，首先检查池中是否有值相同的字符串对象，如果有则不需要创建直接从池中刚查找到的对象引用；如果没有则新建字符串对象，返回对象引用，并且将新创建的对象放入池中。

但是，通过 new 方法创建的 String 对象是不检查字符串池的，而是直接在堆区创建一个新的对象，也不会把对象放入池中，上述原则只适用于通过直接量给 String 对象引用赋值的情况。

注意：

```java
String str = new String("abc")
// 使用 new String 创建的对象不检查常量池，直接在堆去创建一个 String
// 但是 “abc” 这个也是一个 String，创建它时会使用常量池的，有则直接使用，没有则创建并加入
// 也就是说这行代码一个创建了两个 Strin 对象，一个位于常量池中，一个位于堆中
```

String 提供了 intern() 方法。调用该方法时，如果常量池中包括了一个等于此 String 对象的字符串（由 equals 方法确定），则返回池中的字符串。否则，将此 String 对象添加到池中，并且返回此池中对象的引用。

```java
String str1 = "123"; 			 // 通过直接量赋值方式，放入字符串常量池
String str2 = "123"; 
String str3 = new String(“123”); // 通过 new 方式赋值方式，不放入字符串常量池

System.out.println(str1 == str2); // true
System.out.println(str1 == str3.intern()); // true
```

#### 为什么 String 被设计成 final 的

1. 安全，Java 中用到 String 地方非常多，很多的参数、类加载时的类名等等，都是 String 类型的，如果 String 能够改变，那么这些行为将变得不再安全。
2. String 在被多线程共享，并且访问频繁时，可以省略同步和锁等待的时间，进而提高多线程程序的性能。
3. 高效，出于对常量池的考虑，在 Java 中会使用常量池来保存 String，当 2 个 String 对象拥有相同的值时，他们只引用常量池中的同一个拷贝，这样不经能够节省内存，还提高运行效率，如果 String 可变，那么常量池就没法用了。

#### StringBuilder 的扩容

StringBuilder 由它的父类 AbstractStringBuilder 实现，在 AbstractStringBuilder 中，有几个重要的成员：

```java
// 使用一个字符数组来存储 (Java9 使用的是 byte[])
char[] value;
// 使用 count 来表示字符串的长度
int count;
public int length() {
    return count;
}
// 使用 value.length 来表示容量
public int capacity() {
    return value.length;
}
```

通过 append 方法来分析扩容

```java
public AbstractStringBuilder append(String str) {
    if (str == null)         
    	return appendNull();
    // 1. 获取要添加的字符串长度
    int len = str.length();
    // 2. AbstractStringBuilder 扩容，传入当前长度 + 要添加的长度
    ensureCapacityInternal(count + len);
    // 3. 将要添加的字符串 copy 到 value 数组中
    str.getChars(0, len, value, count);
    // 4. 重新计算长度
    count += len;
    return this;
}
```

ensureCapacityInternal 扩容方法

```java
private void ensureCapacityInternal(int minimumCapacity) {
    // 1. 预期的字符串长度比当前的容量大
    if (minimumCapacity - value.length > 0) {
        // 2. newCapacity 重新计算容量
        // 3. Arrays.copyOf 给数组扩容
        value = Arrays.copyOf(value, newCapacity(minimumCapacity));
    }
}

// 重新计算容量
private int newCapacity(int minCapacity) {
    // 1. 新的容量 = 当前容量 * 2 + 2
    int newCapacity = (value.length << 1) + 2;
    // 2. 新的容量小于最小容量，返回最小容量
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    // 3. newCapacity 溢出 Int 的范围 或 超过了 MAX_ARRAY_SIZE，调用 hugeCapacity
    return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
        ? hugeCapacity(minCapacity)
        : newCapacity;
}

private int hugeCapacity(int minCapacity) {
    // 容量溢出 int 的范围，抛出 OutOfMemoryError
    if (Integer.MAX_VALUE - minCapacity < 0) {
        throw new OutOfMemoryError();
    }
    // 没有溢出 int 范围返回 最小容量 和 MAX_ARRAY_SIZE 中较小的值
    return (minCapacity > MAX_ARRAY_SIZE)
        ? minCapacity : MAX_ARRAY_SIZE;
}

// 给数组扩容，内存复制
// char[] original: 原来的数组
// newLength：新的数组长度
public static char[] copyOf(char[] original, int newLength) {
    char[] copy = new char[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```


