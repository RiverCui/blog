---
title: 'JavaScript 中的 Event Loop'
date: 2024-08-12
description: 'Event Loop 的功能与执行流程'
tags: ["JavaScript", "EventLoop"]
---

## Event Loop 是什么

Event Loop 是 JavaScript 实现**异步**的核心机制，解决 JavaScript **单线程**带来的**运行阻塞**的问题。

> JavaScript 之所以是单线程，是因为最初被用来设计处理网页交互时，需要操作 DOM，如果是多线程，一个线程正在修改 DOM，另一个线程也在修改同一个 DOM，会导致冲突和复杂的并发问题。

### 宏任务和微任务

首先看下 [HTML 规范](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)，从中可得知关于 Event Loop 的几个要点：

1. task queue 宏任务队列
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202412231636588.png)

每个 event loop 中有一个或多个 task queue，task queue 是一个集合而不是队列。

这里说 task queue 是一个集合(set)，而不是队列(list)，是因为 event loop 会从 task queue 中抓取第一个可运行的任务，而不是按照队列顺序一个个地出队。

2. microtask queue 微任务队列
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202412231643045.png)
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202412231644002.png)

每个 event loop 有一个 microtask queue，用于存放微任务，microtask 不是 task queue

---

在浏览器中，常见的宏任务、微任务如下：

#### 宏任务

- `setTimeout`
- `setInterval`
- `setImmediate`
- `I/O` 事件
- `MessageChannel`

#### 微任务

- `Promise.then/catch/finally`
- `process.nextTick` (Node.js 环境，优先级高于 Promise)
- `MutationObserver` (监视 DOM 变化)
- `queueMicroTask` (直接创建微任务的方法)

### 一图了解 Event Loop

了解完宏任务、微任务，我们可以看下这张图，在 JavaScript 实际运行的过程中，代码被一行行执行，任务会被加入到执行栈中(call stack)，执行完毕后从栈中弹出。JavaScript 提供的 WebAPIs (例如 `setTimeout`、`DOM`、`fetch` 等)方法会将它们的回调添加到 task queue 中，等待执行。`queueMicrotask`、`Promise`、`MutationObserver` 等方法会创建微任务，添加到 microtask queue 中等待执行。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202412231540040.png)

Event Loop 事件循环在其中负责这几件事：

- 监控调用栈(call stack)
- 管理任务队列(task queue/microtask queue)

Event Loop 事件循环机制保证 JavaScript 能够在单线程环境中有效地进行异步编程，从而提高应用程序的响应性和性能，让我们来看看它的机制具体是怎样的。

## Event Loop 执行流程

Event Loop 的执行流程可以分为以下几个步骤：

1. 执行同步代码
   
   执行当前宏任务中的所有**同步代码**

2. 清空微任务队列
   
   当同步代码执行完毕后，立即检查微任务队列

   执行所有微任务，直到**清空微任务队列**

   如果在清空微任务队列时，产生了新的微任务，也会**立即执行**

3. UI 渲染

   微任务队列清空后，检查是否需要 UI 渲染，如果需要就进行 UI 渲染

4. 执行下一个宏任务

   从宏任务队列中取出**一个**任务执行，重复2-4的步骤

{{< callout emoji="🔥" >}}
   微任务的优先级比宏任务高，在 DOM 渲染前执行，且会一次性执行完毕。宏任务在 DOM 渲染后执行，一次只执行一个宏任务，然后进入下一次事件循环。
{{< /callout >}}

### 例子

理解 Event Loop 最好的方法就是实践，让我们来举个例子，下面这段代码会如何执行：

```JavaScript
console.log('start')

setTimeout(() => {
   console.log('setTimeout')
}, 0)

Promise.resolve().then(() => {
   console.log('promise1')
}).then(() => {
   console.log('promise2')
})

console.log('end')
```

运行结果：
```javascript
start
end
promise1
promise2
setTimeout
```

执行流程分析：
1. print `start`
2. 执行 `setTimeout`，将回调函数注册到宏任务队列
3. 执行 `Promise`，将回调函数注册到微任务队列
4. print `end`
5. Call Stack 清空
6. 执行所有微任务，print `promise1`、`promise2`，微任务队列清空
7. 取宏任务队列中的第一个执行，print `setTimeout`


## See Also
[HTML standard - event loops](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)