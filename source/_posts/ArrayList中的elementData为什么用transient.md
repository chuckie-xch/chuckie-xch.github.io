---
title: ArrayList中的elementData为什么用transient
date: 2017-12-20 10:05:53
tags: Java源码
icon: fa-bolt
---

# ArrayList中elementData数组为什么是transient

## java中的transient关键字

Java中serialization提供了一种持久化实例对象的机制。有可能有个对象的成员我们不想用serialization机制来保存它。就可以在这个成员变量上加上关键字transient。
**transient**是Java中的关键字，用来表示一个域不是该对象串行化的一部分。

## ArrayList中elementData用**transient**的原因
elementData数组是实现ArrayList的关键数据结构。但数组中的元素往往大于我们真实存储的数据，当容量不够时会继续扩容。如果将elementData串行化，会导致空间的浪费。所以ArrayList的设计者将elementData设计为transient,然后再**writeObject**方法中手动进行序列化，只序列互实际存储的那些元素。

```java
   /**
     * Save the state of the <tt>ArrayList</tt> instance to a stream (that
     * is, serialize it).
     *
     * @serialData The length of the array backing the <tt>ArrayList</tt>
     *             instance is emitted (int), followed by all of its elements
     *             (each an <tt>Object</tt>) in the proper order.
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

