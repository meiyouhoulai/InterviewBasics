## Java 并发：ConcurrentHashMap

#### 概述

对与线程安全的 Map，我们常用的有 HashTable、Collections.synchronizedMap、以及 ConcurrentHashMap

Hashtable 和 Collections.synchronizedMap 都是简单粗暴的在 get、put 等方法加上 synchronized，效率很低。

所以对于并发 Map，我们一般首选 ConcurrentHashMap。

#### 实现原理