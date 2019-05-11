---
layout:     post
title:      "同步计数器"
subtitle:   " \"线程阀-多线程的交互\""
date:       2018-06-22 20:44:00
author:     "wmf"
header-img: "img/java.jpg"
catalog: true
tags:
    - 并发
---
## 前言
文章主要学习线程阀--多线程的交互--同步计数器
### 使用场景
需要等到多个条件同时达到才能做下一步操作
场景：由5个线程同时下载，全部下载完才算结束
模拟场景，需要三个程序都干完了才算程序完成
##### CountDownLatch
CountDownLatch是一个同步辅助类，直译过来就是倒计数(CountDown)门闩(Latch)
主要方法
CountDownLatch(int count): 构造一个用给定计数初始化的CountDownLatch
countDown()：递减锁存器的计数
await()：使线程在计数至零前一直等待，除非线程被中断
例子
```java
public class CountDownLatchTest {
    public static void main(String[] args) {
        try {
        CountDownLatch cd = new CountDownLatch(3);
            for (int i=0;i<3;i++) {
                new Thread(new Down(cd)).start();
            }
            cd.await(); //使线程等在这一步，什么时候cd计数变为0什么时候往下走
            System.out.println("all job is finished");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Down implements Runnable {
    private CountDownLatch cd;
    public Down (CountDownLatch cd) {
        this.cd = cd;
    }
    @Override
    public void run() {
        try {
            Thread.sleep(1000); //模拟干活
            System.out.println(Thread.currentThread().getName()+" job is finish");
            cd.countDown(); //递减
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
输出
```
Thread-0 job is finish
Thread-1 job is finish
Thread-2 job is finish
all job is finished
```
如果查看CountDownLatch源码实现，发现它主要使用了继承于AbstractQueuedSynchronizer的Sync，AbstractQueuedSynchronizer是java.util.concurrent的核心组件之一，它为并发提供了一组公共的*基础设置*
<a href="http://www.baidu.com">专门的AQS知识</a>




