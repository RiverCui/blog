---
title: 'JavaScript 中的 Event Loop'
date: 2024-08-12
description: 'Event Loop 的功能与执行流程'
tags: ["JavaScript"]
---

## Event Loop 是什么

Event Loop 是 JavaScript 实现**异步**的核心机制，解决 JavaScript **单线程**带来的**运行阻塞**的问题。

> JavaScript 之所以是单线程，是因为最初被用来设计处理网页交互时，需要操作 DOM，如果是多线程，比如一个线程正在修改 DOM，而另一个线程也在修改同一个 DOM，会导致冲突和复杂的并发问题。

我们先了解下 JavaScript 的执行栈(Call Stack)和任务队列(Callback Queue/Task Queue)，如下图所示：

- 同步代码都会推入到 Call Stack 栈中，代码执行完毕后从栈中弹出。
- 浏览器中的 JavaScript 提供 WebAPIs 例如 setTimeout、DOM、fetch等，这些方法执行后，会将它们的回调添加到 Callback Queue，等待执行。

而 Event Loop 事件循环就是用来监控调用栈(Call Stack)，检查任务队列(Callback Queue)的，来安排执行这些**异步回调代码**。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202412210005052.png)


## Event Loop 执行流程

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

### 宏任务和微任务

实际上，js 中的异步任务分为两类：宏任务(Macro task) 和微任务(Micro tasks)。

微任务的优先级比宏任务高，在 DOM 渲染前执行，且会一次性执行完毕。宏任务在 DOM 渲染后执行，一次只执行一个宏任务，然后进入下一次事件循环。

宏任务：

- `setTimeout`
- `setInterval`
- `setImmediate`
- `I/O` 事件
- `MessageChannel`

微任务：

- `Promise.then/catch/finally`
- `process.nextTick` (Node.js 环境，优先级高于 Promise)
- `MutationObserver` (监视 DOM 变化)
- `queueMicroTask` (直接创建微任务的方法)

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