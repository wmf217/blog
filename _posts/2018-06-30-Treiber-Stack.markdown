---
layout:     post
title:      "Treiber Stack"
subtitle:   " \"Treiber Stack\""
date:       2018-06-22 20:44:00
author:     "wmf"
header-img: "img/java.jpg"
catalog: true
tags:
    - concurrency
---
## introduction
This article mainly introduces Treiber Stack, which is a lock-free stack
### principle
first you will have an AtomicReference object as a top pointer<br/>
when push a new task into this stack, firstly, point the top to the newHead only when
the oldHead is still the oldHead by using CAS<br/>
and popping a task is same as above
this is quite simply, here is the sample
### code
```java
public class ConcurrentStack<E> {
    private AtomicReference<Node<E>> top = new AtomicReference<>();

    public void push(E item) {
        Node<E> newHead = new Node<>(item);
        Node<E> oldHead;
        do {
            oldHead = top.get();
            newHead.next = oldHead;
        } while (!top.compareAndSet(oldHead, newHead));
    }

    public E pop() {
        Node<E> oldHead;
        Node<E> newHead;
        do {
            oldHead = top.get();
            if (oldHead == null)
                return null;
            newHead = oldHead.next;
        } while (!top.compareAndSet(oldHead, newHead));
        return oldHead.item;
    }

    private static class Node<E> {
        public final E item;
        public Node<E> next;
        public Node(E item) {
            this.item = item;
        }
    }
}
```





