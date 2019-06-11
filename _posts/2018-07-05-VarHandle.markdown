---
layout:     post
title:      "VarHandle"
subtitle:   " \"VarHandle\""
date:       2018-07-01 20:44:00
author:     "wmf"
header-img: "img/java.jpg"
catalog: true
tags:
    - concurrency
---
## introduction
this section will show how to use a VarHandle
## main
A VarHandle is a dynamically strongly typed reference to a variable
and it can reference to static fields,non-static fields,
or array elements
---
here is my option on how to use it, first u need a lookup,
which seems like a map for the class, it created as following
way 
```java
class Vh {
    static {
        try {
            MethodHandles.Lookup l = MethodHandles.lookup(); //get a lookup
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
}
```
as we can see, it reference to a class instead of an object, so it seems
it had to be created in a Static method, and by the lookup you can get a
VarHandle reference to a field, also in the static method, so the complete
code are as follows
```java
class Vh {
    public volatile int x;
    public static final VarHandle X;
    static {
        try {
            MethodHandles.Lookup l = MethodHandles.lookup(); //get a lookup
            X  = l.findVarHandle(Vh.class, "x", int.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    Vh (Integer x) {
        this.x = x;
    }
}
```
---
it seems like it is a value (VarHandle) record the offset of a field
relatived to the class, so it is only an offset, not connection with
any object, but it give the power to operate an object`s field by 
atomic, how to do it?
```java
public class VarHandleLearn {
    @Test
    public void test () {
        Vh vh = new Vh(2);
        vh.X.compareAndSet(vh,2,3); //vh is the point
        System.out.println(vh.x);
    }
}
```
it is a typical operate CAS, As above, when use X, the static field
to invoke compareAndSet, the first arg is vh, it is just like the call or apply
in JS, vh is the point on who will compare and set the value '3'
---
it differs from when you want to refer to a static field, as when using compareAndSet
or some other method you do not have to offer a certain object, and there still
is a different when creatting the VarHandle, will use lookup`s method findStaticVarHandle
```java
public class VarHandleLearn {
    @Test
    public void test () {
        Vh vh = new Vh(2);
        Vh.X.compareAndSet(vh,2,3);
        Vh.Y.compareAndSet(8, 10);
        System.out.println(vh.x);
        System.out.println(Vh.y);
    }
}

class Vh {
    public volatile int x;
    public static int y = 8;
    public static final VarHandle X;
    public static final VarHandle Y;
    static {
        try {
            MethodHandles.Lookup l = MethodHandles.lookup(); //get a lookup
            X  = l.findVarHandle(Vh.class, "x", int.class);
            Y  = l.findStaticVarHandle(Vh.class, "y", int.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    Vh (Integer x) {
        this.x = x;
    }
}
```
---
In addition to supporting access to variables in various access modes, VarHandle provides a set of memory fence methods
```java
public static void fullFence() {
    UNSAFE.fullFence();
}
public static void acquireFence() {
    UNSAFE.loadFence();
}
public static void releaseFence() {
    UNSAFE.storeFence();
}
public static void loadLoadFence() {
    UNSAFE.loadLoadFence();
}
public static void storeStoreFence() {
    UNSAFE.storeStoreFence();
}
```
will introduce it later

