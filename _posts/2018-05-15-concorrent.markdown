---
layout:     post
title:      "java并发基础"
subtitle:   " \"java并发基础学习记录\""
date:       2018-05-15 20:44:00
author:     "wmf"
header-img: "img/java.jpg"
catalog: true
tags:
    - 并发
---
#### ThreadGroup
线程组：线程组是为了方便管理出现的，可以同一设定线程组的一些属性，比如setDaemon（设置守护线程）
树结构：在一个进程中线程组是以树形式的方法存在，通常情况下根线程组为system，system下面是main线程组。默认情况下第一级应用自己创建的线程是main线程创建的
#### ThreadLocal
当地线程副本
当使用ThreadLocal维护变量时，ThreadLocal为每一个使用该变量的线程提供一个副本，每个线程副本之间互不影响，提供set，get，remove，initValue方法
定义方法：
```java
private static ThreadLocal<String> a;
```
使用方法
```
....
```
源码实现：
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
解析：
使用ThreadLocal会使用一个ThreadLocalMap来存放不同线程的不同值，其中线程为k，存储的值集合为v，get和set会根据Thread.currentThread()取值
#### synchronized
隐式锁
可以修饰方法和修饰在代码块上
```java
public synchronized void synMethod {}
```
```java
public void synMethod {synchronized(this){}}
```
synchronized(object) object可以是任意对象，上面两种定义方式都是锁定当前对象this
当一个线程拿到一个对象的锁，其它线程对该对象的所有synchronized(object)
阻塞
越小的对象加锁和释放锁的时间越小
#### lock
显式锁
常用方法：lock(),tryLock(),unLock()
lock是一个显式锁的接口
###### ReentrantLock
lock的一个实现类，可支持公平模式和非公平模式，内部使用AQS实现(详见AQS)
###### lock和synchronized比较
lock更灵活，但必须释放锁
lock只可用于代码块锁，synchronized是对象之间互斥
#### ReadWriteLock
也是一个接口，提供了readLock和writeLock两种锁机制
源码：
```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```
*ReadWriteLock与lock是聚合关系，不是继承关系*
适合读多写少的场景
###### ReentrantReadWriteLock
ReadWriteLock的一个实现类，也可以公平和默认的不公平
内部依然用AQS实现(详见AQS)
###### 读写锁的机制
读锁-不让写
写锁-不让读、不让写
*一种读写分离的思想*
#### StampedLock
支持乐观锁和悲观锁(TODO)
#### volatile
不稳定的，易变的
它告诉程序不要试图使用缓存机制，意思是不要使用告诉缓存中的数据，要从内存中直接获取
它能保证可见性，但保证不了原子性
#### atomic
原子操作(要么全部执行成功，要么都别执行，不能被中断)
原理：利用了CPU的比较交换(CAS)，底层调用JNI(允许java调用其它语言，其中CAS是通过java借助c调用CPU底层指令实现的)
#### 单例的实现
第一种：线程不安全，不正确的写法
```java
class Singleton {
    private static Singleton instance;
    private Singleton () {}
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
第二种：线程安全，但是高并发性能不是很高
```java
class Singleton {
    private static Singleton instance;
    private Singleton () {}
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
第三种：线程安全，性能又高，比较常见的写法
```java
class Singleton {
    private static Singleton instance;
    private static byte[] lock = new byte[0];
    private Singleton () {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (lock) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
第四种：也挺好
```java
class Singleton {
    private static Singleton instance;
    private static ReentrantLock lock = new ReentrantLock();
    private Singleton () {}
    public static Singleton getInstance() {
        if (instance == null) {
            lock.lock();
            if (instance == null) {
                instance = new Singleton();
            }
            lock.unlock();
        }
        return instance;
    }
}
```



