---
layout:     post
title:      "浅谈event loop"
subtitle:   ""
date:       2020-07-25
author:     "pompeygao"
header-img: "img/post-bg-2015.jpg"
tags:
    - event loop

---





#### 一、一道常见题目

------

```js
console.log(1)

setTimeout(() => {
    console.log(2)
    new Promise(resolve => {
        console.log(4)
        resolve()
    }).then(() => {
        console.log(5)
    })
})

new Promise(resolve => {
    console.log(7)
    resolve()
}).then(() => {
    console.log(8)
})

setTimeout(() => {
    console.log(9)
    new Promise(resolve => {
        console.log(11)
        resolve()
    }).then(() => {
        console.log(12)
    })
})
```

大家可以先想下输出顺序是什么样的。

答案：

```
1
7
8
2
4
5
9
11
12
```

是否与你想的一致呢？这个题目背后考察的知识点就是 Event Loop。

#### 二、Event Loop

---

JavaScript是单线程语言，它在一个时间点只能处理一键事情，完成一件事情之后才能继续执行另一件事。为了解决这个问题，于是产生了使用异步这种方式来模拟多线程，而支撑异步的就是 Event Loop。

##### 同步任务和异步任务

同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；

异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

具体来说，异步执行的运行机制如下。（同步执行也是如此，因为它可以被视为没有异步任务的异步执行。）

> （1）所有同步任务都在主线程上执行，形成一个[执行栈](http://www.ruanyifeng.com/blog/2013/11/stack.html)（execution context stack）。
>
> （2）主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。
>
> （3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
>
> （4）主线程不断重复上面的第三步。

![event-loop](/img/in-post/event-loop/event-loop.jpg)

##### 任务队列

- `<script src="...">`加载外部脚本时，task 是执行它。
- 用户移动鼠标时，task 是调用`mousemove`事件并执行处理。
- 当调用`setTimeout`时，task 是执行其回调函数。
- …等等。

这些 task 形成一个队列（queue），即 task queue:

![eventLoop](/img/in-post/event-loop/eventLoop.svg)

当引擎正忙于执行时`script`，用户可能会移动鼠标`mousemove`，`setTimeout`可能是由于定时器到期执行，等等，这些任务形成队列，如上图所示。

队列中的任务以“先到先执行”的方式处理。引擎`script`执行完之后，它将处理`mousemove`事件，然后处理`setTimeout`，依此类推。

到目前为止，很容易理解的。

任务队列内任务有两种：`macrotask` 和 `microtask`

##### macrotask（宏任务）

属于 macrotask 的有：`script`、`setTimeout`、`setInterval`、`I/O`、`UI Rendering`、`script当中的所有代码`、`setImmediate(node环境)`

##### microtask（微任务）

![eventLoop-full](/img/in-post/event-loop/eventLoop-full.svg)

属于 microtask 的有: `process.nextTick(node环境)` 、`Promise` 、`MutationObserver`

同步任务执行完后，会执行 microtask ，然后再判断有没有 macrotask ，执行完后再检查有没有 microtask ，如此反复直到执行完毕。

我们分析下开头的题目：

> 同步运行的代码首先输出：`1、7`
>
> 接着，清空microtask任务：`8`
>
> 第一个任务执行：`2、4`
>
> 接着，清空microtask任务：`5`
>
> 第二个任务执行：`9、11`
>
> 接着，清空microtask任务：`12`
>
> 所以执行结果为： `1，7，8，2，4，5，9，11，12`





#### 参考资料

1. [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
2. [Event loop: microtasks and macrotasks](<https://javascript.info/event-loop>)
3. [Javascript 的 Event Loop]([https://medium.com/hobo-engineer/ricky%E7%AD%86%E8%A8%98-javascript-%E7%9A%84-event-loop-c17a0a49d6e4](https://medium.com/hobo-engineer/ricky筆記-javascript-的-event-loop-c17a0a49d6e4))