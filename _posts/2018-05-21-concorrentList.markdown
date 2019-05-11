---
layout:     post
title:      "线程安全集合"
subtitle:   " \"线程安全集合\""
date:       2018-05-21 20:44:00
author:     "wmf"
header-img: "img/java.jpg"
catalog: true
tags:
    - 并发
---
### MAP
###### HashTable
它的使用方法功能和HashMap几乎一样,不同点是它是一个线程安全的散列表，原理如下：
```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}
```
使用synchronized关键字保证了线程的安全
###### ConcurrentHashMap
线程安全版的HashMap，它的效率比较高，主要因为内部分段segment，每段都是一个小HashTable，不同段之间可以并行
### LIST
###### CopyOnWriteArrayList
内部使用ReentrantLock来加锁，读操作不加锁，写操作加锁，写的时候新建一个副本，然后改变引用指向副本，避免读到写了一半的数据
*也就是说如果一个线程正在改，另一个线程正在读，读线程读到的数据和写线程无关，是写线程之前的数据*
###### Vector
矢量队列，通过数组保存数据
与HashTable的线程安全机制类似，都是synchronized
### SET
###### CopyOnWriteArraySet
在CopyOnWriteArrayList基础上使用了java装饰模式，比如add方法调用的是CopyOnWriteArrayList的addIfAbsent方法
```java
public CopyOnWriteArraySet() {
    al = new CopyOnWriteArrayList<E>();
}
public boolean add(E e) {
    return al.addIfAbsent(e);
}
```
### CopyOnWrite的缺点
内存占用大
数据一致性问题(只能保证最终的一致性，不能保证实时一致性)
### StringBuffer
区别于StringBuilder，它是线程安全的




