---
layout:     post
title:      "vue自定义指令"
subtitle:   " \"自己开发过程中使用的自定义指令\""
date:       2017-11-13 20:10:00
author:     "wmf"
header-img: "img/in-post/vue.jpg"
catalog: true
tags:
    - js
    - vue
---

##前言
这篇记录开始时已经很久没写vue代码了，但还是对vue的一些写法记忆比较深刻，记忆最深的是vue的核心理念：以数据为核心和双向绑定,很不幸现在的工作还是大多使用jquery，不过我还是在这种情况下，通过babel编译在工作中学习并使用es6
##正文
自定义指令概述：除了默认设置的核心指令( v-model 和 v-show ), Vue 也允许注册自定义指令
####直接贴代码
* 长按指令 longtouch.js
```js
export default {
    bind (el, binding, vnode) {
        let num = 0
        let timer = ''
        function clickHandler (e) {
            timer = setTimeout(function () {
                if (e.which == 1) {
                    num++
                    if (num == 2) {
                        num = 0
                        if (binding.expression) {
                            binding.value(e)
                        }
                    }
                    clickHandler(e)
                }
            }, 100)
        }
        function removeHandler (e) {
            clearTimeout(timer)
        }
        el.__vueMousedown__ = clickHandler
        el.__vueMouseout__ = removeHandler
        el.addEventListener('mousedown', el.__vueMousedown__)
        el.addEventListener('mouseup', el.__vueMouseout__)
        el.addEventListener('mouseout', el.__vueMouseout__)
    },
    unbind (el, binding) {
        el.removeEventListener('mousedown', el.__vueMousedown__)
        el.removeEventListener('mouseup', el.__vueMouseout__)
        el.removeEventListener('mouseout', el.__vueMouseout__)
    }
}
```
* 点击元素以外指令 clickoutside.js
```js
export default {
    bind (el, binding, vnode) {
        function documentHandler (e) { //  当前点击元素
            if (el.contains(e.target)) { //  查看当前点击的dom对象是否在el之内
                return false
            }
            if (binding.expression) { // 绑定的方法名
                binding.value(e) // 运行绑定函数，e可以省略，但最好把点击的元素给传出去
                // 所有外面绑定的函数可以有一个参数也可以没有
            }
        }
        el.__vueClickOutside__ = documentHandler //  给el绑定一个方法
        document.addEventListener('mouseover', documentHandler) //  document添加监听事件
    },
    unbind (el, binding) {
        document.removeEventListener('mouseover', el.__vueClickOutside__)
        delete el.__vueClickOutside__
    }
}
```
其中长按指令是自己开发出来的所以重点记录下，还有很多常用指令网上可以copy直接用，有时间再补