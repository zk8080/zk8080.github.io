---
title: JS事件循环(Event Loop)
date: 2019-04-11 21:12:42
categories: 前端
tags:
    - JavaScript
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;日常开发中，总会出现各种因为同步异步行为导致的问题。最近和同事们讨论React中setState同步异步的问题，在网上看到一些解析和总结，发现自己其实对js的执行机制并不太理解。因此在网上查阅了一些资料，来完善一下自己的知识，本文是自己记录一下对js执行机制的理解。
先看一段代码:
```
    setTimeout(function(){
        console.log('a')
    },0)
    console.log(b)
    setTimeout(function(){
        console.log(c)
    },1000)
    console.log(d)
    setTimeout(function(){
        console.log(e)
    },0)
```
按照js是一门单线程语言，代码是从上到下的顺序执行的，我们会很正常的认为打印的结果是`abdec`,但是去chrome中验证下，却发现结果并不是这样。  
原来js中将代码执行分为了同步和异步，当同步代码执行完毕后，才会去执行异步代码。`setTimeout、setInterval、callback、Promise`等，都属于异步操作。但是具体的执行机制是什么呢？我们可以看下面这张图(出处在文章尾部):
![引用地址在文章结尾](http://pp0hf9wwd.bkt.clouddn.com/15fdd88994142347)
从这张图里我们可以知道:
+ js会将任务分为同步任务和异步任务，同步任务进入主线程，异步任务进入Event Table，并且注册回调函数，
- 回调函数进入Event Queue;
* 主线程的任务执行完毕，到Event Queue中读取对应的函数，然后进入主线程去执行。
以上会重复执行，这就是js的执行机制，也是事件循环(Event Loop)。  

知道了这些后，再看上面的代码，我们知道了`setTimeout`属于异步操作，会进入Event Table,等待主线程任务执行完毕才会去执行。但是`setTimeout(function(){},0)`是不是会立即呢？答案是不会。`setTimeout(fn, 0)`是指当主线程任务执行完毕后会立即执行，所以我们会看到这段代码的结果是`bdaec`。