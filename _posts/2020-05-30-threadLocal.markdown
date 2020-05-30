---
layout:     post
title:     "threadLocal的内存泄露问题"
subtitle:   " \"threadLocal的内存泄露问题\""
date:       2020-05-30 22:44:00
author:     "wmf"
header-img: "img/in-post/bigdata.jpg"
catalog: true
tags:
    - java
---
## 对象的引用方式
强软弱虚四种
## 强引用
普通的写法即强引用
```java
Object obj = new Object()
```
当GC回收时，拥有强引用的对象不会不清楚，及时内存不足，出现OOM事件，也不会清楚
## 软引用
```java
Object aRef = new Object();
SoftReference aSoftRef=new SoftReference(aRef); 
```
GC回收时，弱引用对象也不会被回收，但当内存不足时，弱引用对象就会被回收掉，可以通过get获取，适合当做缓存使用
## 弱引用
```java
WeakReference<People>reference=new WeakReference<People>(new People("zhouqian",20));，作用另说
```
弱引用的特点，只要经历一次GC就会被直接回收掉，可以通过get获取
## 虚引用
最弱的引用，甚至连get都get不到，但虚引用指向的对象在被GC回收时会收到系统通知，它的实际用处是JVM用来清理堆外内存使用的(堆外内存不归GC管，但JVM需要通过软引用在堆内存被GC时接受通知以确定有没有相应的堆外内存可回收)，实际开发中用不到
## 弱引用的用处
新建一个threadLocal变量
```java
public static void main(String[] args) {
    ThreadLocalTest threadLocalTest = new ThreadLocalTest();
    threadLocalTest.a.set("王二狗");
    System.out.println(threadLocalTest.a.get());
}
public static class ThreadLocalTest {
    public ThreadLocal<String> a = new ThreadLocal<String>();
}
```
ThreadLocal线程当地副本，set的源码如下
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}
```
可以看到，ThreadLocal为每个线程创建了一个map，其中map的key就是上述代码的a，value就是set里的参数，之所以是个map是因为可能会有很多个ThreadLocal变量，如a，b，c(如果不用ThreadLocal，a.b.c本身也指向不同的内存空间可以理解成一个map)<br>
由于a=new ThreadLocal<String>()，所以a实际上指向了堆中新开辟出来的这个空间，那么当a在本地线程map中作为key时，这个key也要指向这个内存空间，那么问题来了，当我们把a=null时实际意义是要释放这段内存空间，但由于线程本地map中依然有引用指向该内存空间，所以a=null并不能如预期那样把这段内存空间释放出来<br>
这个时候弱引用就发挥它的作用了，查看ThreadLocal的源码
```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
```
其中Entry就是一个弱引用
```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
也就谁说ThreadLocal中的key是通过弱引用指向a指向的内存空间的，所以当a=null时，这段内存空间由于只有弱引用指向它，经过一次GC直接就被清除了，key自动变为null，达到了预期的效果
## ThreadLocal内存泄露
以上所说，当a=null，指向空间被GC回收，本地线程map中的key自动变为空，但是value中的如果是一个指向某Object的引用，那么这个Object还是有强引用指向，由于key已经变为null，永远不可能访问到value，那么value指向的内存空间就无法被GC清除，造成内存泄露
## 解决方案 
为了解决这个问题，只要a=null之前执行a.remove()就可以解决















