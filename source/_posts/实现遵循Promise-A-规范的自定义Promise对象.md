---
title: 实现遵循Promise/A+规范的自定义Promise对象
date: 2019-11-14 17:58:58
categories: 前端
tags:
    - JavaScript
---
最近几天看了一些关于promise的文章，自己也根据理解总结一下，怎么手写一个符合Promise/A+规范的Promise。
# 什么是Promise
阮一峰老师的《ECMAScript 6 入门》中是这样解释的：
>Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。
# 什么是Promise/A+规范
>一个开放的，可实现的，可互操作的JavaScript Promise的标准。

具体的规范可以看一下官方文档：
+ [英文原文文档](https://promisesaplus.com/)
- [中文翻译文档](https://juejin.im/post/5c4b0423e51d4525211c0fbc)

# 实现一个简单的Promise
根据文档我们知道Promise有几条基本要求：
+ Promise是一个构造函数，接受一个回调函数为参数，并且该回调函数有两个函数类型的参数resolve、reject
- Promise共有三种状态：pengding、fulfilled、rejected，但Promise的当前状态只能是这三种状态中的一种，并且状态只能由pending转换为fulfilled/rejected这两种状态，改变为fulfilled/rejected状态后不可再改变。
* 状态改变只能通过调用resolve、reject两个函数进行。
+ 需要提供一个then方法去访问当前值或者返回结果，then方法接受两个函数类型参数：onFulfilled、onRejected。
- onFulfilled：当Promise状态为fulfilled时进行调用，并且接受Promise成功的值为参数。
* onRejected：当Promise状态为rejected时进行调用，并且接受Promise失败的值为参数。
+ then方法返回一个新的Promise

根据这些要求我们可以先实现一个简易版的Promise：
```js
    const PENDING = "pengding";
    const FULFILLED = "fulfilled";
    const REJECTED = "rejected";

    function MyPromise(fn) {
        const self = this;
        self.status = PENDING;
        self.value = null;
        self.reason = null;

        function resolve(value) {
            if (self.status === PENDING) {
                self.status = FULFILLED;
                self.value = value;
            }
        }

        function reject(reason) {
            if (self.status === PENDING) {
                self.status = REJECTED;
                self.reason = reason;
            }
        }

        fn(resolve, reject);
    }

    MyPromise.prototype.then = function (onFulfilled, onRejected) {
        const self = this;
        let bridgePromise = null;
        if (self.status === FULFILLED) {
            return bridgePromise = new MyPromise((resolve, reject) => {
                try {
                    let x = onFulfilled(self.value);
                    resolve(x);
                } catch (e) {
                    reject(e)
                }
            })
        }

        if (self.status === REJECTED) {
            return bridgePromise = new MyPromise((resolve, reject) => {
                try {
                    let x = onRejected(self.reason);
                    resolve(x);
                } catch (e) {
                    reject(e)
                }
            })
        }
    }
```
我们实现了这个简易的Promise，但是可以发现这个Promise是不支持异步操作的，所以接下来我们按照Promise/A+的规范，实现一个符合规范的Promise。

# 实现符合Promise/A+规范的Promise   
根据规范，我们上面实现的简易版，除了不能进行异步操作外，还缺少了一个resolvePromise函数去处理then函数中的回调函数的返回值。所以接下来我们一步步来实现。
## 增加异步操作
首先我们需要定义两个保存回调函数的数组，分别保存onFulfilled回调函数和onRejected回调函数，并且在执行resolve或reject后，进行调用回调函数。
```js
    function MyPromise(fn) {
        const self = this;
        self.status = PENDING;
        self.value = null;
        self.reason = null;
        // 存放onFulfilled回调处理函数集合
        self.onFulfilledCallbacks = [];
        // 存放onRejected回调处理函数集合
        self.onRejectedCallbacks = [];

        function resolve(value) {
            if (self.status === PENDING) {
                self.status = FULFILLED;
                self.value = value;
                // 修改状态后执行回调函数集合
                self.onFulfilledCallbacks.forEach((callback) => callback(value));
            }
        }

        function reject(reason) {
            if (self.status === PENDING) {
                self.status = REJECTED;
                self.reason = reason;
                //执行reject的回调函数，将reason传递到callback中
                self.onRejectedCallbacks.forEach((callback) => callback(reason));
            }
        }

        fn(resolve, reject);
    }
```
然后修改一下then方法，需要加一个pengding状态的判断，将对应的回调函数，加入到回调函数集合中：
```js
    if(self.status === PENDING){
        return bridgePromise = new MyPromise(function(resolve, reject){
            self.onFulfilledCallbacks.push(value => {
                try{
                    let x = onFulfilled(value);
                    resolve(x);
                }catch(e){
                    reject(e)
                }
            })

            self.onRejectedCallbacks.push(reason => {
                try{
                    let x = onRejected(reason);
                    resolve(x);
                }catch(e){
                    reject(e)
                }
            })
        })
    }
```
经过我们的改善，我们实现的简易版Promise已经支持异步操作了，接下来我们实现resolvePromise这个方法。

## 实现resolvePromise函数，处理回调函数结果
resolvePromise函数是Promise/A+规范中，规定对Promise结果进行解析的处理程序，其中判断了多种Promise返回结果的情况，具体判断规则可以参考文档，这里我们来根据文档实现这个函数：
```js
    function resolvePromise(bridgePromise, x, resolve, reject) {
        /**
        * 2.3.1 如果返回的 bridgePromise 和 x 是指向同一个引用（循环引用），则抛出错误
        */
        if(bridgePromise === x){
            return reject(new TypeError('循环调用'));
        }
        /**
        * 2.3.2 如果 x 是一个 promise 实例，则采用它的状态：
        * 2.3.2.1 如果 x 是 pending 状态，那么保留它（递归执行这个 promise 处理程序），直到 pending 状态转为 fulfilled 或 rejected 状态
        * 2.3.2.2 如果或当 x 状态是 fulfilled，resolve 它，并且传入和 promise1 一样的值 value
        * 2.3.2.3 如果或当 x 状态是 rejected，reject 它，并且传入和 promise1 一样的值 reason
        */

        if(x instanceof MyPromise){
            if(x.status === PENDING){
                x.then(y => {
                    resolvePromise(bridgePromise, y, resolve, reject);
                }, error => {
                    reject(error);
                })
            }else{
                x.then(resolve, reject);
            }
        }else if(x != null && ((typeof x === 'object') || (typeof x === 'function'))){
            /**
            * 2.3.3 此外，如果 x 是个对象或函数类型
            * 2.3.3.1 把 x.then 赋值给 then 变量
            * 2.3.3.2 如果捕获（try，catch）到 x.then 抛出的错误的话，需要 reject 这个promise
            * 2.3.3.3 如果 then 是函数类型，那个用 x 调用它（将 then 的 this 指向 x）,第一个参数传 resolvePromise ，第二个参数传 rejectPromise：
            * 2.3.3.3.1 如果或当 resolvePromise 被调用并接受一个参数 y 时，执行[[Resolve]](promise, y)
            * 2.3.3.3.2 如果或当 rejectPromise 被调用并接受一个参数 r 时，执行 reject(r)
            * 2.3.3.3.3 如果 resolvePromise 和 rejectPromise 已经被调用或以相同的参数多次调用的话吗，优先第一次的调用，并且之后的调用全部被忽略（避免多次调用）
            * 2.3.3.4 如果 then 执行过程中抛出了异常，
            * 2.3.3.3.4.1 如果 resolvePromise 或 rejectPromise 已经被调用，那么忽略异常
            * 2.3.3.3.4.2 否则，则 reject 这个异常
            * 2.3.3.4 如果 then 不是函数类型，直接 resolve x（resolve(x)）
            */
            // 判断是否已经调用的标识
            let called = false;
            try {
                let then = x.then;
                if(typeof then === 'function'){
                    then.call(x, y => {
                        if(called) return;
                        called = true;
                        resolvePromise(bridgePromise, y, resolve, reject);
                    },error => {
                        if(called) return;
                        called = true;
                        reject(error);
                    })
                }else{
                    resolve(x);
                }
            } catch (error) {
                if(called) return;
                called = true;
                reject(error);
            }

        }else{
            resolve(x);
        }

    }

```
resolvePromise函数已经实现，接下来我们完善一下then方法：
```js
    MyPromise.prototype.then = function (onFulfilled, onRejected) {
        const self = this;

        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason};

        let bridgePromise = null;

        if(self.status === FULFILLED){
            return bridgePromise = new MyPromise(function(resolve, reject){
                setTimeout(() => {
                    try {
                        let x = onFulfilled(self.value);
                        resolvePromise(bridgePromise, x, resolve, reject)
                    } catch (error) {
                        reject(error)
                    }
                }, 0);
            })
        }

        if(self.status === REJECTED){
            return bridgePromise = new MyPromise(function(resolve, reject){
                setTimeout(() => {
                    try {
                        let x = onRejected(self.reason);
                        resolvePromise(bridgePromise, x, resolve, reject)
                    } catch (error) {
                        reject(error)
                    }
                }, 0);
            })
        }

        if(self.status === PENDING){
            return bridgePromise = new MyPromise(function(resolve, reject){
                self.onFulfilledCallbacks.push(value => {
                    try {
                        let x = onFulfilled(value);
                        resolvePromise(bridgePromise, x, resolve, reject)
                    } catch (error) {
                        reject(error)
                    }
                })

                self.onRejectedCallbacks.push(reason => {
                    try {
                        let x = onRejected(reason);
                        resolvePromise(bridgePromise, x, resolve, reject)
                    } catch (error) {
                        reject(error)
                    }
                })
            })
        }
    }
```
完善后的then方法中，你会发现我们多加了一些代码：
1. 增加了onFulfilled和onRejected函数的判断，没有传入函数则给一个自定义函数，返回之前resolve的值，或者抛出异常，这也是Promise/A+的规范所要求的。
2. 在fulfilled和rejected的状态返回值中都包了一层setTimeout，因为Promise的then方法都需要在下一轮的事件循环中被异步调用，所以我们模拟使用了setTimeout，原生的Promise则是使用微任务实现的，而pengding状态中没有加setTimeout，则是要在resolve和reject函数中进行异步调用回调函数集合,这是为了满足Promise/A+规范的2.2.4和2.2.6。

到此，我们已经把then方法完整的实现了，接下来我们将整个代码进行整合，然后进行测试。

# 符合Promise/A+规范的完整代码
```js
const PENDING = 'pengding';
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function MyPromise(fn) {
    const self = this;
    self.status = PENDING;
    self.value = null;
    self.reason = null;
    self.onFulfilledCallbacks = [];
    self.onRejectedCallbacks = [];


    function resolve(value) {
        if (value instanceof MyPromise) {
            return value.then(resolve, reject);
        }

        if(self.status === PENDING){
            setTimeout(() => {
                self.value = value;
                self.status = FULFILLED;
                self.onFulfilledCallbacks.forEach((callback) => callback(self.value));
            }, 0);
        }
    }

    function reject(reason) {
        if(self.status === PENDING){
            setTimeout(() => {
                self.reason = reason;
                self.status = REJECTED;
                self.onRejectedCallbacks.forEach((callback) => callback(self.reason));
            }, 0);
        }
    }

    try {
        fn(resolve, reject);
    }catch(e){
        reject(e);
    }
}

function resolvePromise(bridgePromise, x, resolve, reject) {
    if(bridgePromise === x){
        return reject(new TypeError('循环调用'))
    }

    if(x instanceof MyPromise){
        if(x.status === PENDING){
            x.then(y => {
                resolvePromise(bridgePromise, y, resolve, reject);
            }, error => {
                reject(error);
            })
        }else{
            x.then(resolve, reject);
        }
    }else if(x != null && ((typeof x === 'object') || (typeof x === 'function'))){
        let called = false;
        try {
            let then = x.then;
            if(typeof then === 'function'){
                then.call(x, y => {
                    if(called) return;
                    called = true;
                    resolvePromise(bridgePromise, y, resolve, reject);
                },error => {
                    if(called) return;
                    called = true;
                    reject(error);
                })
            }else{
                resolve(x);
            }
        } catch (error) {
            if(called) return;
            called = true;
            reject(error);
        }

    }else{
        resolve(x);
    }

}

MyPromise.prototype.then = function (onFulfilled, onRejected) {
    const self = this;
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason};

    let bridgePromise = null;

    if(self.status === FULFILLED){
        return bridgePromise = new MyPromise(function(resolve, reject){
            setTimeout(() => {
                try {
                    let x = onFulfilled(self.value);
                    resolvePromise(bridgePromise, x, resolve, reject)
                } catch (error) {
                    reject(error)
                }
            }, 0);
        })
    }

    if(self.status === REJECTED){
        return bridgePromise = new MyPromise(function(resolve, reject){
            setTimeout(() => {
                try {
                    let x = onRejected(self.reason);
                    resolvePromise(bridgePromise, x, resolve, reject)
                } catch (error) {
                    reject(error)
                }
            }, 0);
        })
    }

    if(self.status === PENDING){
        return bridgePromise = new MyPromise(function(resolve, reject){
            self.onFulfilledCallbacks.push(value => {
                try {
                    let x = onFulfilled(value);
                    resolvePromise(bridgePromise, x, resolve, reject)
                } catch (error) {
                    reject(error)
                }
            })

            self.onRejectedCallbacks.push(reason => {
                try {
                    let x = onRejected(reason);
                    resolvePromise(bridgePromise, x, resolve, reject)
                } catch (error) {
                    reject(error)
                }
            })
        })
    }
}

// 执行测试用例需要用到的代码
MyPromise.deferred = function() {
    let defer = {};
    defer.promise = new MyPromise((resolve, reject) => {
        defer.resolve = resolve;
        defer.reject = reject;
    });
    return defer;
}
try {
    module.exports = MyPromise
} catch (e) {}

```

代码已经整合完毕，我们可以利用Promise的测试脚本，对我们的代码进行测试：
```
npm i promises-aplus-tests -g
promises-aplus-tests promise.js
```