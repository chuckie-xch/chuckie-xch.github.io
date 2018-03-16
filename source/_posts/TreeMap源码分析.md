---
title: TreeMap源码分析
date: 2018-03-16 23:39:02
tags: JDK源码
---

`TreeMap`是基于红黑树实现的。并且实现了`SortedMap`接口，是可排序的，如果构造器没有传入比较器，则根据键按自然排序。`TreeMap`并不是线程安全的，如果高并发写的情况下需要开发人员自行实现同步，或者通过以下代码，包装成一个同步的`SortedMap`：

```java
SortedMap map = Collections.synchronizedSortedMap(new TreeMap());
```

<!-- more -->

## TreeMap的类结构图：

[![TreeMap-Class-Structure.png](https://i.loli.net/2018/03/16/5aabc82119ae3.png)](https://i.loli.net/2018/03/16/5aabc82119ae3.png)

## 成员变量

```java
// Red-black mechanics

private static final boolean RED   = false;
private static final boolean BLACK = true;
private final Comparator<? super K> comparator;
private transient Entry<K,V> root;
private transient int size = 0;
private transient int modCount = 0;
```

`TreeMap`中有个Entry内部类，封装了红黑数的数据结构

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;
    ...
}
```

## 核心方法解析

**左旋：**

```java
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> r = p.right;
        p.right = r.left;
        if (r.left != null)
            r.left.parent = p;
        r.parent = p.parent;
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;
        r.left = p;
        p.parent = r;
    }
}
```

**右旋：**

```java
private void rotateRight(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> l = p.left;
        p.left = l.right;
        if (l.right != null) l.right.parent = p;
        l.parent = p.parent;
        if (p.parent == null)
            root = l;
        else if (p.parent.right == p)
            p.parent.right = l;
        else p.parent.left = l;
        l.right = p;
        p.parent = l;
    }
}
```

左旋和右旋的代码很简单，看不懂的请先去看看红黑树剖析一文。

下面分析下TreeMap中的最核心的一些查找方法，快速查找，才是TreeMap的灵魂。先来看看TreepMap中都有哪些查找方法：

[![TreeMap中的查找方法.png](https://i.loli.net/2018/03/16/5aabcbee1caee.png)](https://i.loli.net/2018/03/16/5aabcbee1caee.png)

先从`V get(Object key)`这个方法看起：

```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}
```

**注意：**

* 当指定的key不能被比较时，会抛出`ClassCastException`
* 当`TreeMap`采用key自然排序并且指定key为`Null`，或者比较器不允许`Null`值时，会抛出`NullPointerException`

```java
final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}
```

通过二叉树查找，遍历结点，查找和key值相等Entry，如果找不到，返回Null。



* 插入方法put()

put(K key, V value)方法是将指定的键值对添加到map里，如果key已经存在，则替换原来的value，返回oldValue，如果不存在，则在红黑数中查找到合适的位置进行插入，如果插入后破坏了红黑树规则，则进行调整。

```java
public V put(K key, V value) {
    ......
    int cmp;
    Entry<K,V> parent;
    if (key == null)
        throw new NullPointerException();
    Comparable<? super K> k = (Comparable<? super K>) key;//使用元素的自然顺序
    do {
        parent = t;
        cmp = k.compareTo(t.key);
        if (cmp < 0) t = t.left;//向左找
        else if (cmp > 0) t = t.right;//向右找
        else return t.setValue(value);
    } while (t != null);
    Entry<K,V> e = new Entry<>(key, value, parent);//创建并插入新的entry
    if (cmp < 0) parent.left = e;
    else parent.right = e;
    fixAfterInsertion(e);//调整
    size++;
    return null;
}
```

接下来看看关键的`fixAfterInsertion()`方法：

```java
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;

    while (x != null && x != root && x.parent.color == RED) {
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    root.color = BLACK;
}
```