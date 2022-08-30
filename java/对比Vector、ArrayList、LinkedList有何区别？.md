## 对比 Vector、ArrayList、LinkedList 有何区别？

> 2019/5/15

### 概述

Vector 底层是数组数据结构，线程安全，因为效率低，被 ArrayList 替代了，开发几乎用不到，
多线程访问时使用 Collections.synchronizedList(new ArrayList())。

ArrayList 底层的数据结构使用的是数组结构，特点：查询快，增删慢，非线程安全。

LinkedList 底层使用的链表数据结构，特点：增删快，查询慢，非线程安全。

### ArrayList 的扩容

#### ensureCapacityInternal

```java
private void ensureCapacityInternal(int minCapacity) {
  if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
    minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
  }

  ensureExplicitCapacity(minCapacity);
}
```

minCapacity 的初始值 size + 1

elementData 是当前 ArrayList 中存储数据的数组，如果 elementData 为空，去 DEFAULT_CAPACITY 和 minCapacity 中的较大的值。

#### ensureExplicitCapacity 

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

minCapacity ( size + 1) 大于 elementData 的长度进行扩容

#### grow 真正的扩容方法

```java
private void grow(int minCapacity) {
  // overflow-conscious code
  int oldCapacity = elementData.length;
  int newCapacity = oldCapacity + (oldCapacity >> 1);
  if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
  if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
  // minCapacity is usually close to size, so this is a win:
  elementData = Arrays.copyOf(elementData, newCapacity);
}
```

新的容量扩充到旧容量的 1.5 倍，之后进行数据拷贝。