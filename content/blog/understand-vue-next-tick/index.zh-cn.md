---
title: '理解 Vue.js nextTick'
date: 2024-08-16
description: "什么是 Vue.js nextTick，如何实现以及怎样使用。"
tags: ["Vue"]
---

`nextTick` 是 Vue 提供的方法，用于在下一次 DOM 更新循环结束后执行延迟回调，主要用于获取更新后的 DOM 状态。因为 **Vue 在更新 DOM 时是异步执行**的，因此在数据变化后，视图不会立即更新，会把所有的 DOM 更新操作放入到一个队列中，等下一次事件循环(event loop)结束后统一执行。

`nextTick` 的实现依赖于 `event loop` 机制，如果你不知道 `event loop`，可以先看我这篇文章了解下 [JavaScript 中的 Event Loop](https://www.moon-odyssey.com/event-loop-in-javascript/)。

## nextTick 使用方法、场景

Vue 提供了 `nextTick` 的全局 API，在 Vue2 中可以直接通过 `this.$nextTick()` 调用，在 Vue3 中，`nextTick` 作为一个独立函数，可以直接从 Vue 包中导入并使用。

```javascript
// vue2
this.$nextTick(() => {
  // ...
})

// vue3
import { nextTick } from 'vue';

nextTick(() => {
  // ...
})
```

`nextTick` 主要用于获取更新后的 DOM 状态，异步操作 DOM，下面是常见的使用场景：

- 修改数据后需要立即获取 DOM

```javascript
this.message = 'updated message'
this.$nextTick(() => {
  console.log('message', this.$el.textContent)
})
```

- 也可以与 `Promise` 结合使用

```javascript
async updateMessage() {
  this.message = 'updated message'
  await this.$nextTick()
  console.log('message', this.$el.textContent)
}
```

- 在生命周期钩子中进行 DOM 操作

在 `created()` 钩子中，DOM 尚未渲染完毕，因此任何 DOM 操作都可能无效。将这些操作放在 `nextTick()` 中以确保在 DOM 渲染后执行。

## nextTick 实现

`nextTick` 核心实现思路：

- 队列机制

  `nextTick` 的回调函数会被放到一个任务队列，不会立即执行。当 DOM 更新完后，队列中的任务会被一次性执行。

- 优先使用微任务

  vue2 会先优先使用 `Promise` (微任务)，同时也做了降级处理，用来适配更多浏览器，这个下文会详细说。
  vue3 默认浏览器支持 `Promise`(微任务)。

### Vue2 优雅降级

伪代码：
```javascript
let callbacks = []  // 任务队列，存储通过 nextTick 提交的回调函数
let pending = false  // 标志位，防止重复启动任务调度

// 用于执行当前任务队列中的所有回调
function flushCallbacks() {
  pending = false  // 清除标志，表示任务已被执行
  const copies = callbacks.splice(0)  // 复制当前任务队列
  callbacks.length = 0  // 清空任务队列
  for (let cb of copies) {
    cb()  // 执行每个回调
  }
}

let timerFunc  // 动态分配的调度函数，根据不同的运行环境选择不同的实现方式，宏任务或微任务
if(typeof Promise !== 'undefined') {  // 优先使用 Promise，创建微任务
  timerFunc = () => Promise.resolve().then(flushCallbacks)
} else if(typeof MutationObserver !== 'undefined') {  // 回退到 MutationObserver，通过监听 DOM 变化触发回调，创建微任务
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode('')  // 创建一个伪 DOM 节点
  observer.observe(textNode, {characterData: true})
  timerFunc = () => {
    textNode.data = String(Math.random())  // 每次改变其内容时触发回调
  }
} else if(typeof setImmediate !== 'undefined') {  // 回退到 setImmediate，创建宏任务
  timerFunc = () => setImmediate(flushCallbacks)  // 如果环境支持 setImmediate(edge 和 IE)
} else {  // 最终回退到 setTimeout
  timerFunc = () => setTimeout(flushCallbacks, 0)
}

export function nextTick(cb, ctx) {
  callbacks.push(() => {
    if(cb) {
      cb.call(ctx)  // 确保回调的上下文绑定
    }
  })
  if(!pending) {
    pending = true  // 标记为已调度
    timerFunc()  // 调用调度方法
  }
}
```

vue2 的调度机制优先级如下：

1. `Promise` 微任务
2. `MutationObserver` 微任务
3. `setImmediate` 宏任务(仅edge、IE支持)
4. `setTimeout` 宏任务

### Vue3 Promise

与 vue2 相比，vue3 更加简洁和现代化，完全依赖于 `Promise` 微任务机制。

伪代码：
```javascript
// 创建一个状态为 fulfilled 的 Promise实例
// 用于调度微任务，在同步代码执行完成后立即进行微任务队列
const resolvedPromise = Promise.resolve()

// 用来跟踪当前是否有正在进行的刷新操作
// 如果存在未完成的刷新任务，nextTick 会返回相同的 Promise，以确保回调在适当的时机执行
let currentFlushPromise: Promise<void> | null = null

export function nextTick<T = void>(this: T, fn?: (this: T) => void): Promise<void> {  // fn 是可选的回调函数
  const p = currentFlushPromise || resolvedPromise  // 如果当前有未完成的刷新任务，使用这个 Promise，否则使用 resolvedPromise
  return fn ? p.then(() => fn.call(this)) : p  // 如果传入了回调函数 fn，通过 Promise 调用它，否则仅返回 Promise
}
```
