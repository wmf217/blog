---
layout:     post
title:      "JS原型和原型链"
subtitle:   " \"原型\""
date:       2017-02-24 20:10:00
author:     "wmf"
header-img: "img/in-post/vue.jpg"
catalog: true
tags:
    - js
    - 面向对象
---

##前言
在js语言种不存在class的概念，在js种想要实现面向对象的思想是通过构造函数(constructor)和原型链(prototype chains)实现的
##正文
#### 构造函数
实际上就是一个function而已
```js
var A = function (name) {
    this.name = name;
    this.getName = function () {
        return this.name;
    }
}
var a = new A ();
var b = new A ();
``` 
构造函数种定义了一个属性一个方法，这样可以正常实现类的定义和实例化，但是有一个问题如下
```js
console.log(a.getName === b.getName);//false
```
从上面可以看出每次new一个实例出来都新创建了一个getName方法，而这种显然不是想要的结果，而且耗费资源，这样的一个函数我们还是希望可以实现对象的共享，于是乎就出现了prototype
#### 原型(prototype)
*为了解决实例无法共享属性的问题，js提供了prototype*
稍微改造下上面代码
```js
var A = function (name) {
    this.name = name;
}
A.prototype.getName = function () {
    return this.name;
}
var a = new A ();
var b = new A ();
console.log(a.getName === b.getName);//true
``` 
通过chrome查看一下对象实例a的结构
```js
var a = new A ('wmf');
console.log(a);
```
如下图
![](/img/in-post/prototype.png)
*通过构造函数生成对象实例时，会将对象实例的原型指向构造函数的prototype属性。每一个构造函数都有一个prototype属性，这个属性就是对象实例的原型对象*
以上的代码，构造函数A的prototype属性相当于保存了一个对象，结构大概是
```js
{
    getName: function () {
        return this.name;
    },
    constructor: function () {
        ......
    },
    _proto_: Object
}
```
其中getName是之前通过prototype添加的属性，constructor属性默认指向该构造函数(这个我是暂时用不到)，当然这个对象也是有原型，原型指向一个Object对象(原型链的最顶端)，在实例化a对象时a的原型_proto_指向了该对象，如果再次实例化对象b，那么a,b的原型同时指向该对象。实现了属性的共享。
这也是为什么之前原型链继承会出现(原型中引用类型属性的共享)
#### 原型对象属性共享
```js
var A = function (name) {
	this.name  = name;
	this.arr   = [1,2,3];
	this.getIn = function () {
		console.log(this.arr);
	}
}
A.prototype.getOut = function () {
    console.log(this.name);
};
var c1 = new A ('wmf');
var c2 = new A ('qly');
A.prototype.getOut = function () {
	console.log(this.arr);
};
c1.getOut(); //[1,2,3]
c1.getIn();  //[1,2,3]
```


