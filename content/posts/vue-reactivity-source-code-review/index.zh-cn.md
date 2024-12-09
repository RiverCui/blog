+++
title = 'Vue Reactivity Source Code Review'
date = 2024-11-27T14:36:46+08:00
draft = true
+++

#TODO
- Vue2 响应式原理
- 缺点
- Vue3 改进

# Vue2响应式原理

在 Vue2 中，当一个 JavaScript 对象传入 Vue 实例作为 `data` 选项时，Vue 会遍历此对象所有的 `property`，并使用 **`Object.defineProperty()`** 把这些 `property` 转为 `getter/setter`。在 `getter` 中收集依赖，在 `setter` 中触发更新：每个组件实例都对应一个 `watcher` 实例，它会在组件的渲染过程中把接触过的数据 `property` 记录为依赖，当依赖项的 `setter` 触发时，会去通知 `watcher`，从而使他关联的组件重新渲染。这就是 Vue2 响应式实现的思路，至于具体是如何实现的，下文将会展开分析。

![](https://v2.cn.vuejs.org/images/data.png)

## Object.defineProperty()

`Object.defineProperty()` 是 Vue 实现响应式的基础，它允许精确地添加或修改对象上的属性，语法如下：

```javascript
Object.defineProperty(obj, prop, descriptor)
```
参数：
`obj`: 要定义属性的对象
`prop`: 要定义或修改的属性名
`descriptor`: 要定义或修改的属性描述符

`descriptor` 常用属性：
- `enumerable`：属性是否可枚举，默认为 false
- `configurable`：属性是否可被修改或删除，默认为 false
- `get`：获取属性的方法
- `set`：设置属性的方法

*参考：[MDN-Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty#syntax)*

## 实现响应式
在 Vue 源码中，通过 `defineReactive` 函数对 `Object.defineProperty` 进行封装,实现响应式。`get` 会触发 `reactiveGetter` 实现依赖收集，`set` 会触发 `reactiveSetter` 通知依赖更新。下面是这个函数的源码的简化版本，只需要看非注释部分：

参数：
- `obj`（需要绑定的对象）
- `key`（对象的属性名）
- `val`（具体的值）

```javascript
function defineReactive(obj, key, val) {
  // 创建依赖收集器
  // const dep = new Dep()
  
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      // 依赖收集
      // if (Dep.target) {
      //     dep.depend()
      // }
      return val
    },
    set: function reactiveSetter(newVal) {
      if (newVal === val) return
      val = newVal
      // 通知所有依赖
      // dep.notify()
    }
  })
}
```
在本篇文章开头提到，Vue 会遍历对象的所有属性`property`，所以现在还需要再封装一层 `Observer` 去遍历对象，将一个普通 JavaScript 对象的所有属性转换成响应式的。简化版本如下：

```javascript
class Observer {
  constructor(value) {
    // 给被观察的对象添加 __ob__ 属性，指向这个 Observer 实例
    def(value, '__ob__', this)
    if(Array.isArray(value)) {
      // 劫持数组方法
      // ...
      // 处理数组
      this.observeArray()
    } else {
      // 处理对象
      const keys = Object.keys(value)
      for (let i = 0; i < keys.length; i++) {
        const key = keys[i]
        defineReactive(value, key, NO_INITIAL_VALUE, undefined, shallow, mock)
      }
    }
  }
  observeArray(value) {
    for (let i = 0, l = value.length; i < l; i++) {
      observe(value[i], false, this.mock)
    }
  }
}
```

其中，`def()` 是一个工具函数，用来给对象添加属性：
```javascript
export function def(obj, key, val) {
  Object.defineProperty(obj, key, {
    value: val,
  })
}
```

`observe()` 是入口函数，会返回 Observer 实例：
```javascript
function observe(value) {
  if (typeof value !== 'object') return
  let ob
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else {
    ob = new Observer(value)
  }
  return ob
}
```

## 依赖收集
刚才 `defineReactive` 中被注释掉的几行代码，也就是下文中高亮的部分：在 `get` 时进行依赖收集，在 `set` 时通知所有依赖，这是实现响应式系统的另一个重要部分——**依赖收集**。

```javascript {linenos=table, hl_lines=["2-3" "9-12" "18-19"]}
function defineReactive(obj, key, val) {
  // 创建依赖收集器
  const dep = new Dep()
  
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      // 依赖收集
      if (Dep.target) {
          dep.depend()
      }
      return val
    },
    set: function reactiveSetter(newVal) {
      if (newVal === val) return
      val = newVal
      // 通知所有依赖
      dep.notify()
    }
  })
}
```

### 为什么需要依赖收集
举个例子，现在有这么个 Vue 对象：
```javascript
new Vue({
  template:`
    <div>
      <span>{{ text1 }}</span>
      <span>{{ text2 }}</span>
    </div>
  `,
  data: {
    text1: 'text1',
    text2: 'text2',
    text3: 'text3',
  }
})
```
要对 `text3` 进行修改：
```javascript
data.text3 = 'text3 modify'
```
虽然修改了 `data` 中 `text3` 的值，但是由于模板渲染中并没有用到 `text3`，所以并不会触发上文中的 `dep.notify()` 去通知依赖、更新视图。

再举个例子，假如现在有个全局对象，而且我们在两个 Vue 对象中使用了它。
```javascript
let globalObj = {
  text1: 'text1'
}

let o1 = new Vue({
  template:`
    <div><span>{{ globalObj.text1 }}</span></div>
  `,
  data: {
    globalObj
  }
})

let o2 = new Vue({
  template:`
    <div><span>{{ globalObj.text1 }}</span></div>
  `,
  data: {
    globalObj
  }
})
```
这个时候修改 `globalObj.text1` 的值：
```javascript
globalObj.text1 = 'text1 modify'
```
在响应式系统中，当 `globalObj.text1` 的值变化，应该通知 `o1`、`o2` 这两个 vm 实例去更新视图。**依赖收集**就是来实现这个的：依赖收集会让这个数据(`globalObj.text1`)知道，有两个地方(`o1`、`o2`)依赖自身，所以当这个数据(`globalObj.text1`)变化时，会去通知依赖自身的 `o1`、`o2`。


### 依赖收集的核心 —— Dep 和 Watcher

让我们回归代码，看看具体是如何实现依赖收集的。

首先需要一个 `Dep`，每个 `Dep` 实例有一个数组去存放 watchers，当数据变化时会通知 watchers 更新。同样的，每个 `Watcher` 实例也有一个数组去存放它依赖的 deps。也就是说，**依赖收集是双向的**，Dep 记录 Watcher，Watcher 也记录 Dep，这个很重要。

思维导图.png
多对多，响应式对象 Object 中存在 Dep，Dep 中存放多个 Watcher，Watcher 负责通知依赖更新。

先看看 `Dep`：
```javascript
class Dep {
  static target  // 静态属性，全局唯一的当前正在执行的 Watcher
  id  // 实例属性，每个 dep 的唯一标识
  subs  // 实例属性，存储所有订阅者

  constructor {
    this.id = uid++
    this.subs = []
  }

  // 添加订阅者
  addSub(sub) {
    this.subs.push(sub)
  }

  // 移除订阅者
  removeSub(sub) {
    remove(this.subs, sub)
  }

  // 建立依赖关系
  depend() {
    if(Dep.target) { // 如果有正在执行的 Watcher，将这个 Watcher 作为依赖
      Dep.target.addDep(this)
    }
  }

  // 通知所有订阅者
  notify() {
    for(let i = 0, l = this.subs.length; i < l; i++) {
      this.subs[i].update()
    }
  }
}

Dep.target = null;
```

如上所述，`class Dep` 的 target 作为静态属性，用来记录当前唯一正在执行的 Watcher，`id` 用来给 Dep 实例添加唯一标识，`subs` 用来存储所有订阅者。`addSub()` 和 `removeSub()` 用来添加订阅者、移除订阅者，`depend()` 用来建立依赖关系，`notify()` 用来通知所有的订阅者。

`Watcher` 会复杂一点，让我们来看一下：

```javascript
class Watcher {
  constructor (vm, expOrFn, cb, options={}) {
    if(options) {
      this.lazy = !!this.lazy
      this.sync = !!this.sync
    }

    this.vm = vm  // Vue 实例
    this.cb = cb  // 回调函数
    this.deps = []  // 存储该 watcher 依赖的所有 dep
    this.newDeps = []  // 新一轮依赖收集的 dep
    this.depIds = new Set()  // 依赖的id集合，用于去重
    this.newDepIds = new Set()  // 新依赖收集的id集合
    this.expression = expOrFn.toString()  // 用于调试
    this.dirty = this.lazy
    this.id = ++uid  // 唯一标识

    // 这里定义了 getter
    if(typeof expOrFn === 'function') {
      this.getter = expOrFn  // 如果 expOrFn 是函数，直接作为 getter
    } else {
      this.getter = parsePath(expOrFn) // 如果是字符串，转换成获取对象属性的函数
    }

    // 立即执行一次 getter，进行初始化和依赖收集
    this.value = this.lazy ? undefined : this.get()
  }

  get() {
    pushTarget(this)  // 将当前 watcher 设置为全局活动的 watcher
    let value
    try {
      // 执行 getter，触发被观察对象的 getter，从而收集依赖
      value = this.getter.call(this.vm, this.vm)
    } catch (e) {
      throw e
    } finally {
      popTarget()  // 恢复之前活动的watcher
      this.cleanupDeps()  // 清理依赖
    }
    return value
  }

  addDep(dep) {
    const id = dep.id
    // 检查是否已在新的依赖集合中
    if(!this.newDepIds.has(id)) {
      // 添加到新的依赖 id 集合中
      this.newDepIds.add(id)
      // 添加到新的依赖数组中
      this.newDeps.push(dep)
      // 检查是否在旧依赖中
      if(!this.depIds.has(id)) {
        // 如果不在旧依赖中，让 dep 去收集当前 watcher
        dep.addSub(this)
      }
    }
  }

  // 清理依赖
  cleanupDeps() {
    let i = this.deps.length
    // 遍历旧依赖数组
    while (i--) {
      const dep = this.deps[i]
      // 如果新依赖数组中不包含这个旧依赖
      if (!this.newDepIds.has(dep.id)) {
        // 将这个 watcher 从 dep 中移除
        dep.removeSub(this)
      }
    }
    // depIds 更新，清空 newDepIds
    let tmp: any = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    // deps 更新，清空 newDeps
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  update() {
    // lazy 模式，懒计算，不会立即重新计算，只是将 dirty 标记为 true
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {  // sync 模式，同步更新模式，立即更新
      this.run()
    } else {  // 默认异步更新模式，将 watcher 放入更新队列
      queueWatcher(this)
    }
  }

  run() {
    const value = this.get()  // 获取新的值
    if(value !== this.value || isObject(value) || this.deep) {  // 如果新旧值不相等 | 新值是对象 | 深度监听
      const oldValue = this.value  // 保存旧值
      this.value = value  // 设置新值
      this.cb.call(this.vm, value, oldValue)  // 执行回调函数，并传入新值和旧值
    }
  }
}
```

在 `Watcher` 的 `constructor` 中，会先处理传入的 `expOrFn`，当它是函数时，会直接赋给 `getter`，如果是字符串，那么还需要 `parsePath` 来转为访问函数，例如 `parsePath('user.name')` 会返回函数：`(obj) => obj['user']['name']`，注意此处 `obj` 就是 `getter.call` 中调用到的 `vm`，也就是当前 Vue 实例。此外还会立即执行一次 `getter()`，进行初始化和依赖收集。

看下面的代码，为了处理嵌套的 Watcher 场景，在 `getter()` 开始时，会将当前 `Watcher` 也就是 `target` 推入 targetStack 栈中，并将 `Dep.target` 指向当前 `Watcher`，待 `getter()` 结束时，又会将这个 `Watcher` 从栈中弹出，将 `Dep.target` 的指向修改为上一个 `Watcher`。

```javascript
const targetStack = []

export function pushTarget(target) {
  targetStack.push(target)
  Dep.target = target
}

export function popTarget() {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}
```

而 `Watcher` 的 `addDep` 方法，和 `Dep` 中的 `addSub` 方法互相调用，做双向依赖收集的处理，不用担心死循环的问题，因为 `Watcher` 还有一个 `cleanupDeps` 来及时的清理依赖。

`update()` 和 `dep()` 则负责更新处理，当数据变化时执行相应的操作。先看 `update()`，在数据变化时，它有三种处理方式：
1. `lazy` 模式
   ```javascript
   if(this.lazy) {
    this.dirty = true
   }
   ```
   
   在源码中搜索 `dirty` 可以找到 `evaluate()` 方法：
   ```javascript
   /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
   evaluate() {
     this.value = this.get()
     this.dirty = false
   }
   ```

   当 `evaluate()` 被调用时，会调用 `get()` 方法，并将 `dirty` 设为 `false`。那么 `evaluate()` 又在哪里被使用呢？
   全局搜素后，能看到在 `state.ts` 文件中被调用：

   ```javascript
   function createComputedGetter(key) {
     return function computedGetter() {
       const watcher = this._computedWatchers && this._computedWatchers[key]
       if (watcher) {
         if (watcher.dirty) {
           watcher.evaluate()
         }
         if (Dep.target) {
           // ...
           watcher.depend()
         }
         return watcher.value
       }
     }
   }
   ```

  也就是说，当 `computed` 的 `getter()` 被调用时，如果这个 watcher 是脏数据(`dirty` 为 `true`)，那么就会执行 `evaluate()`，使用 `lazy` 模式不会立即重新计算值，只是将 `dirty` 标记为 `true`，等到下次访问这个属性时才会真正计算。
  计算属性 `computed` 就是采用这个模式。

2. `sync` 模式
   ```javascript
   else if(this.sync) {
    this.run()
   }
   ```
   如果是 `sync` 模式，会调用 `run()`：
   ```javascript
   run() {
     const value = this.get()  // 获取新的值
     if(value !== this.value || isObject(value) || this.deep) {  // 如果新旧值不相等 | 新值是对象 | 深度监听
       const oldValue = this.value  // 保存旧值
       this.value = value  // 设置新值
       this.cb.call(this.vm, value, oldValue)  // 执行回调函数，并传入新值和旧值
     }
   }
   ```
   能够看到，`run` 方法会立即执行 `get()`，并且如果新旧值不相等或新值是对象或深度监听的时候，会执行回调函数，并传入新值和旧值。
   所以 `sync` 模式主要是用于需要立即响应数据变化的场景，不过 `sync` 默认是 `false`，大多数情况都会使用 `async` 异步更新。

3. `queue` 异步更新
   ```javascript
   else {
    queueWatcher(this)
   }
   ```
   看下 `queueWatcher()` 做了些什么：
   ```javascript
   let has = {}
   /**
     * Push a watcher into the watcher queue.
     * Jobs with duplicate IDs will be skipped unless it's
     * pushed when the queue is being flushed.
   */
   export function queueWatcher(watcher: Watcher) {
     // 重复检查，防止同一个 watcher 被重复添加到队列中
     const id = watcher.id
     if (has[id] != null) {
       return
     }
     
     // 防止 watcher 在自己的更新过程中触发自己的递归更新
     if (watcher === Dep.target && watcher.noRecurse) {
       return
     }
   
     has[id] = true
     if (!flushing) {  // 如果队列还没有开始刷新，推入队列
       queue.push(watcher)
     } else {
       // 如果队列正在刷新，需要按照 id 顺序插入
       // 确保 watcher 按照创建顺序(id大小)执行，因为：1.父组件的 watcher 要在子组件之前更新 2.computed 要在普通 watcher 之前更新
       let i = queue.length - 1
       while (i > index && queue[i].id > watcher.id) {
         i--
       }
       queue.splice(i + 1, 0, watcher)
     }
     // queue the flush
     if (!waiting) {
       waiting = true
       // 确保队列会在下一个 tick 被刷新，
       nextTick(flushSchedulerQueue)
     }
   }
   ```
   - 当 `flushSchedulerQueue` 被执行时，`flushing` 会被设置为 `true`。
   - 通过 `nextTick()` 调用 `flushSchedulerQueue()`，在下一个**微任务**中执行 `flushSchedulerQueue()`，这样多次修改数据，只会触发一次更新。


## Vue 响应式系统

通过上面的章节我们知道，Vue 的响应式系统的实现，靠的是**数据响应式化**以及**依赖收集**。数据响应式化的核心/原理是 `Object.defineProperty`，它会注册 `get` 和 `set` 进行依赖收集，具体是在响应式对象中 new 一个 Dep 实例来进行依赖收集的处理，将当前的 Watcher 添加到 Dep 实例的订阅者列表(subs)中。那么其实到这里，对于整个响应式系统构建的流程还是有一些模糊的，尤其是 `new Watcher()`的时机，所以当新建一个 Vue 实例时，上面所述的这些处理都是何时进行的呢？

首先我们要知道 Vue 实例中有哪些 Watcher。作为 Vue 用户经常能够接触到的无非就是渲染 Watcher(Render Watcher)、计算属性 Watcher(Computed Watcher)和用户 Watcher(User Watcher)：

```javascript
// 模板渲染 Render Watcher
<div>{{ message }}</div>

// computed 选项，Computed Watcher
computed: {
  total() {
    return this.price * this.quantity
  }
}

// watch 选项，User Watcher
watch: {
  price(newVal, oldVal) {
    console.log('price change:', newVal)
  }
}
```

现在我们不着急关注何时 `new Watcher`，先了解下 `new Vue()` 的整体流程:

```javascript {linenos=table, hl_lines=["14-15" "20-23"]}
function initMixin(Vue) {
  Vue.prototype._init = function () {
    const vm = this
    // 初始化生命周期
    initLifeCycle(vm)
    // 初始化事件
    initEvent(vm)
    // 初始化渲染
    initRender(vm)
    // 调用 beforeCreate Hook
    callHook(vm, 'beforeCreate')
    // 初始化 inject
    initInject(vm)
    // 初始化 state
    initState(vm)
    // 初始化 provide
    initProvide(vm)
    // 调用 created Hook
    callHook(vm, 'created')
    // 如果有 el 选项，自动挂载
    if(vm.$options.el) {
      vm.$mount(vm.$options.el)  // 创建 Render Watcher
    }
  }
}
```

我们主要关注 `initState` 和 `$mount`。

### initState

`initState` 会依次初始化 `props` -> `methods` -> `data` -> `computed` -> `watch`：

```javascript {linenos=table, hl_lines=["10-13" "15-16" "18-21"]}
function initState(vm) {
  const opts = vm.$options
  
  // 1. 初始化 props
  if (opts.props) initProps(vm, opts.props)
  
  // 2. 初始化 methods
  if (opts.methods) initMethods(vm, opts.methods)
  
  // 3. 初始化 data
  if (opts.data) {
    initData(vm)  // 生成响应式对象
  }
  
  // 4. 初始化 computed
  if (opts.computed) initComputed(vm, opts.computed)
  
  // 5. 初始化 watch
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```
**生成响应式对象**的过程，也就是前面提到的 `Object.defineProperty`，就是在 `initData` 中实现：

```javascript
function initData() {
  observe(data)  // 上文中提到的生成响应式对象入口函数
}
```

`initComputed` 和 `initWatch` 分别创建 Computed Watcher 和 User Watcher：

```javascript
function initComputed() {
  // 为每个计算属性创建 computed watcher
  new Watcher(vm, getter, null, options)
}
function initWatch() {
  // 为每个 watch 创建 user watcher
  new Watcher(vm, key, cb, options)
}
```

### $mount

Render Watcher 会在组件挂载时(`$mount`)创建。

```javascript
// 创建渲染 watcher (mount 阶段)
Vue.prototype.$mount = new Watcher(vm, updateComponent, null, options)
```

# Vue2 响应式系统的缺陷

Vue2 的响应式并不是完美的，`Object.defineProperty` 这种实现方式自带一些缺陷，所以在使用 Vue2 时需要注意这些问题。

## 对象

Vue2 无法检测**对象属性的添加和删除**。

例如：

```javascript
var vm = new Vue({
 data: {
   a: 1
 }
})
// vm.a 是响应式的
vm.b = 2
// vm.b 是非响应式的
```

因为 Vue 会在初始化实例时对 property 执行 getter/setter 转化，所以 property 必须在 `data` 对象上存在才能让 Vue 将它转为响应式的。

```javascript
function initData(vm) {
  let data = vm.$options.data
  observe(data)  // 在这里调用 observe, 使对象响应式化
}
```

### 解决方案1 `Vue.set`

对于已经创建的实例，虽然 Vue 不允许动态添加**根级别**的响应式 property。但是可以使用 `Vue.set(object, propertyName, value)` 的方法向嵌套对象添加响应式 property。

例如在 data 中有这么一个对象：

```javascript
var vm = new Vue({
  data: {
    someObject: {
      a: 1,
    }
  }
})
```
使用 `Vue.set` 给 `someObject` 新增键名为 `b` 的响应式 property：

```javascript
Vue.set(vm.someObject, 'b', 2)
```
或

```javascript
this.$set(this.someObject, 'b', 2)
```

### 解决方案2 `Object.assign`

有时需要为已存在的对象赋值多个新 property，可以使用 `Object.assign`。

```javascript
this.someObject = Object.assign({}, this.someObject, {a: 1, b: 2})
```

但要注意，像下面这样新增的 property，是没有响应式的：

```javascript
Object.assign(this.someObject, {a: 1, b: 2})
```

原因很简单，`Object.assign(this.someObject,{a:1,b:2})` 这种写法相当于直接修改、添加属性：

```javascript
// 等同于
this.someObject.a = 1  // 触发 a 的 setter
this.someObject.b = 2  // b 仍为非响应式
```

而 `Object.assign({},this.someObject,{a:1,b:2})` 这种写法可以触发 `someObject` 的 `setter`，会递归遍历 `someObject` 上的所有属性，进行响应式转换。


## 数组

Vue2 无法检测**数组索引**和**长度变化**。

- 数组索引：`vm.items[indexOfItem] = newValue`
- 数组长度：`vm.items.length = newLength`

以上两种变化无法触发响应式更新，因为数组的索引实际上就是对象的属性，虽然理论上 Vue 可以像处理对象属性一样处理数组索引，但是数组过长时，比如一个10000长度的数组，如每个索引都设置 getter/setter，会有严重的**性能问题**。同样的，数组的 `length` 属性也没有做响应式处理，因为可能会引发连锁反应（修改 `length` 可能会影响大量素）。

Vue2 为了解决这个问题，重写了数组的七个方法，这七个方法可以触发响应式更新：

```javascript
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

所以想要触发数组的响应式，应该这样做：

```javascript
// 修改数组索引
vm.items.splice(indexOfItem, 1, newValue)
// 修改数组长度
vm.items.splice(2)
```

## 性能

Vue2 的响应式实现需要**递归遍历**对象的所有属性，本身性能开销就比较大，这也是 Vue3 改为通过 `proxy` 来实现响应式的原因。


# Vue3做了哪些改进

Vue3 的响应式系统基于 ES6 的 `proxy`，是对于 Vue2 中 `Object.defineProperty` 的重大升级，优点有：
1. 可以检测对象属性的添加和删除
2. 可以监听数组变化而无需额外处理
3. 不需要深度递归遍历，性能更好
4. 支持 Map、Set 等数据结构

## proxy 实现响应式

```javascript
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const res = Reflect.get(target, key, receiver)
      track(target, key)  // 依赖收集
      return res
    },
    set(target, key, value, receiver) {
      const res = Reflect.set(target, key, value, receiver)
      trigger(target, key)  // 触发更新
      return res
    }
  })
}
```

这里的 `Reflect` 是一个内置对象，提供拦截 JavaScript 操作的方法。

```javascript
Reflect.get(target, key)  // 获取属性
Reflect.set(target, key, value)  // 设置属性
Reflect.has(target, key)  // 检查属性
Reflect.deleteProperty(target, key)  // 删除属性
```
Reflect 提供统一的操作对象 API，在 proxy 中使用有很多好处：

```javascript {linenos=table, hl_lines=["4-5" "10-11"]}
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      // 使用 Reflect 可以确保 this 的正确指向
      const res = Reflect.get(target, key, receiver)
      track(target, key)
      return res
    },
    set(target, key, value, receiver) {
      // 返回布尔值表示操作是否成功
      const res = Reflect.set(target, key, value, receiver)
      trigger(target, key)
      return res
    }
  })
}
```

## 响应式 API

### ref 和 reactive

### computed

### watch 和 watchEffect


# 源码
本文对源码做了精简处理，只保留了最核心的部分，其实在阅读源码的过程还学到了很多，比如对 removeDep 的优化，提升性能。这些细节不能在此篇文章中一一列举，如果感兴趣可以。