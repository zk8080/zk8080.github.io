---
title: JS事件循环(Event Loop)
date: 2019-04-11 21:12:42
categories: 前端
tags:
    - JavaScript
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;日常开发中，总会出现各种因为同步异步行为导致的问题。最近和同事们讨论React中setState同步异步的问题，在网上看到一些解析和总结，发现自己其实对js的执行机制并不太理解。因此在网上查阅了一些资料，来完善一下自己的知识，本文是自己记录一下对js执行机制的理解。
先看一段代码:
```js
    setTimeout(function(){
        console.log('a')
    },0)
    console.log('b')
    setTimeout(function(){
        console.log('c')
    },1000)
    console.log('d')
    setTimeout(function(){
        console.log('e')
    },0)
```
按照js是一门单线程语言，代码是从上到下的顺序执行的，我们会很正常的认为打印的结果是`abdec`,但是去chrome中验证下，却发现结果并不是这样。
## 同步和异步  
原来js中将代码执行分为了同步和异步，当同步代码执行完毕后，才会去执行异步代码。`setTimeout、setInterval、callback、Promise`等，都属于异步操作。但是具体的执行机制是什么呢？我们可以看下面这张图(出处在文章尾部):
![引用地址在文章结尾](https://raw.githubusercontent.com/zk8080/blog-picture/master/img/event-loop2.jpg)
从这张图里我们可以知道:
+ js会将任务分为同步任务和异步任务，同步任务进入主线程，异步任务进入Event Table，并且注册回调函数，
- 回调函数进入Event Queue;
* 主线程的任务执行完毕，到Event Queue中读取对应的函数，然后进入主线程去执行。
以上会重复执行，这就是js的执行机制，也是事件循环(Event Loop)。  

知道了这些后，再看上面的代码，我们知道了`setTimeout`属于异步操作，会进入Event Table,等待主线程任务执行完毕才会去执行。但是`setTimeout(function(){},0)`是不是会立即呢？答案是不会。因为`setTimeout(fn, 0)`是指当主线程任务执行完毕后会立即执行，不需要再等待多少秒后再执行，所以我们会看到这段代码的结果是`bdaec`。

接着我们再看一段代码:
```js
    setTimeout(function(){
        console.log('a')
    },0)
    new Promise(function(resolve){
        console.log('b')
        resolve()
    }).then(res => {
        console.log('c')
    })
    console.log('d')
```
按照我们上面的同步异步的执行机制，我们可以知道这段代码会进行以下操作：
+ 异步代码`setTimeout`进入Event Table
- 执行同步代码`Promise`，打印b
* `Promise.then`进入Event Table
+ 执行同步代码，打印d
- 主线程执行完毕，到Event Queue中读取对应的函数，并放到主线程中执行

按照之前的理解，这个时候应该是先打印a再打印c，但是我们到chrome中验证一下发现结果是先打印c再打印a，这是为什么呢？

## 宏任务和微任务
查阅了一些文章后，原来js将任务分为了更精细的宏任务(macro-task)和微任务(micro-task): 
+ 宏任务：包括整体代码的script、setTimeout、setInterval
- 微任务：Promise、process.nextTick(暂不了解,后续学习)

宏任务和微任务之间的执行顺序又是什么呢？我们看下面这张图(出处在文章尾部):
![引用地址在文章结尾](https://raw.githubusercontent.com/zk8080/blog-picture/master/img/cfbd39ddcbf42a5bc18725ea0560dc10.png)
从这张图里我们可以知道：
+ js将代码分为宏任务和微任务，先进入宏任务中执行
- 宏任务执行结束后，查看是否有可执行的微任务
* 有微任务就执行微任务，然后进行下一个宏任务
+ 没有微任务则直接进行下一个宏任务

那么再看我们之前的代码，我们就可以明白为什么结果是先打印c再打印a:
+ 整段代码作为宏任务进入主线程
- `setTimeout`进入Event Table，然后注册回调函数，分发宏任务到Event Queue
+ `new Promise`立即执行，打印b，`then`进入Event Table, 注册回调函数，分发微任务到Event Queue
* `console.log()`立即执行，打印d
+ 到此第一个宏任务执行完毕，然后查看是否有微任务
- 有微任务`then`函数，则执行`then`函数，打印c
* 微任务执行完毕，开始执行新的宏任务`setTimeout`的回调函数,打印a

由此我们就了解到js的执行机制。

## 结语
本文是在看到一些文章后，自己所理解的js执行机制，如有不对，请指出。写完这篇文章后，自己对js的事件循环也有了一个大概的了解，虽然具体的原理和底层的堆、栈等还一知半解，但是后续会努力学习加深自己的理解。

## 参考文章
> [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)