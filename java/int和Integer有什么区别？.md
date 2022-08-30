## int 和 Integer有什么区别？

> 2019/5/15

### 概述

int 是我们常说的整型数据，是 Java 的 8 个原始数据类型之一；

Integer 是 int 对应的包装类，提供了一些 int 的基本信息封装和操作，比如 int 的最大值最小值、字符串转 int 等等。

int 和 Integer 之前进行运算，有一个重要的特性：自动装箱和拆箱。

###自动装箱和拆箱

```java
Integer x = 4; // 自动装箱，Integer.valueOf(4)，小心 x = null 运算前进行判断
x = x + 2;     // Integer.valueOf(x.intValue() + 2)，int 与 Interger 运算时，Interger 先自动拆箱

Integer m = 128;
Integer n = 128;
System.out.println(m == n);  // false
Integer a = 127;
Integer b = 127;
Integer c = new Integer(127);
System.out.println(a == b);  // true
System.out.println(a == c);  // false 使用 new 关键字创建出来的 int 和其他比较永远是 false
```

### valueOf

Integer.valueOf 方法是自动装箱的实现方法

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

IntegerCache.low 的值是 -128，IntegerCache.high 的值默认为 127，可以看到在这两个值之间直接从缓存中获取 Integer，不在的话 new 一个 Integer。

默认情况下，IntegerCache.cache 是一个为 256 大小的 Integer 数组，里面的赋值是从 -128 到 127

```java
cache = new Integer[(high - low) + 1];
int j = low;
for(int k = 0; k < cache.length; k++) {
    cache[k] = new Integer(j++);
}       
```

IntegerCache.cache[i + (-IntegerCache.low)] 刚好通过下标取到与 i 值相同的元素。

实时上，IntegerCache.high 只是默认值为 127，这个值我们可以通过实际情况调整的，看下面的源码：

```java
String integerCacheHighPropValue =
    sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
if (integerCacheHighPropValue != null) {
    try {
        int i = parseInt(integerCacheHighPropValue);
        i = Math.max(i, 127);
        // Maximum array size is Integer.MAX_VALUE
        h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
    } catch( NumberFormatException nfe) {
        // If the property cannot be parsed into an int, ignore it.
    }
}
high = h;
```

JVM 提供了修改这个值的参数设置

```java
-XX:AutoBoxCacheMax=N
```

### intValue

intValue 是 Interher 自动拆箱的方法，Interger 使用一个 int value 变量保存 int 装箱后的值，intValue 就是返回这个 value 变量

```java
// 注意这个值是 private final 类型，就是为了防止 Integer 的值被改变
// 如果这个值能改变，会给产品的可靠性带来严重的问题
private final int value;
public int intValue() 
    return value;
}
```

### 其他

- 自动装箱和拆箱发生在编译阶段，由 javac 完成
- 类似 IntegerCache 的缓存不仅仅是 Integer 才有
  - Boolean，缓存了 true/false 对应实例，Boolean.TRUE/FALSE
  - Short，同样缓存了 -128~127 之间的数值
  - Byte，缓存了全部数值
  - Character，缓存范围`\u0000` 到 `\u007F`
- 能使用 int，则避免使用 Integer
- int 不是线程安全的，如果要保证线程安全，推荐使用 AtomicInteger、AtomicLong。