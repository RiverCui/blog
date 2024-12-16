---
title: 'Vue Reactivity Source Code Review'
date: 2024-06-20
description: "Analysis of Vue's reactivity system source code"
tags: ["Vue"]
---

> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

## Vue 2 Reactivity Principles

In Vue 2, when a JavaScript object is passed into a Vue instance as a `data` option, Vue will traverse all `properties` of this object and use **`Object.defineProperty()`** to convert these properties into `getters/setters`. Dependencies are collected in the `getter` and updates are triggered in the `setter`: each component instance corresponds to a `watcher` instance, which records data `properties` accessed during component rendering as dependencies. When a dependency's `setter` is triggered, it notifies the `watcher`, causing its associated component to re-render. This is the approach behind Vue 2's reactivity implementation. As for how it is specifically implemented, we'll analyze this in detail below.

![](https://v2.cn.vuejs.org/images/data.png)

### Object.defineProperty()

`Object.defineProperty()` is the foundation of Vue's reactivity implementation. It allows precise addition or modification of properties on an object. The syntax is as follows:

```javascript
Object.defineProperty(obj, prop, descriptor)
```
Parameters:
`obj`: The object on which to define the property
`prop`: The name of the property to be defined or modified
`descriptor`: The descriptor for the property being defined or modified

Common properties of `descriptor`:
- `enumerable`: Whether the property is enumerable, defaults to false
- `configurable`: Whether the property can be modified or deleted, defaults to false
- `get`: Method for getting the property
- `set`: Method for setting the property

*Reference: [MDN-Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty#syntax)*

Let me continue with the implementation part.

### Implementing Reactivity
In Vue's source code, the `defineReactive` function wraps `Object.defineProperty` to implement reactivity. `get` triggers `reactiveGetter` to collect dependencies, while `set` triggers `reactiveSetter` to notify dependencies to update. Below is a simplified version of this function's source code, just focus on the non-commented parts:

Parameters:
- `obj` (object to bind)
- `key` (object property name)
- `val` (specific value)

```javascript
function defineReactive(obj, key, val) {
  // Create dependency collector
  // const dep = new Dep()
  
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      // Collect dependencies
      // if (Dep.target) {
      //     dep.depend()
      // }
      return val
    },
    set: function reactiveSetter(newVal) {
      if (newVal === val) return
      val = newVal
      // Notify all dependencies
      // dep.notify()
    }
  })
}
```
As mentioned at the beginning of this article, Vue traverses all properties of an object, so we need to wrap another layer called `Observer` to traverse the object and convert all properties of a regular JavaScript object into reactive ones. Here's a simplified version:

```javascript
class Observer {
  constructor(value) {
    // Add __ob__ property to the observed object, pointing to this Observer instance
    def(value, '__ob__', this)
    if(Array.isArray(value)) {
      // Intercept array methods
      // ...
      // Process array
      this.observeArray()
    } else {
      // Process object
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

Here, `def()` is a utility function used to add properties to an object:
```javascript
export function def(obj, key, val) {
  Object.defineProperty(obj, key, {
    value: val,
  })
}
```

`observe()` is the entry function that returns an Observer instance:
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
### Dependency Collection
The previously commented out lines in `defineReactive`, highlighted below, are for dependency collection in `get` and notification of all dependencies in `set` - this is another important part of implementing the reactivity system - **dependency collection**.

```javascript {linenos=table, hl_lines=["2-3" "9-12" "18-19"]}
function defineReactive(obj, key, val) {
  // Create dependency collector
  const dep = new Dep()
  
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      // Collect dependencies
      if (Dep.target) {
          dep.depend()
      }
      return val
    },
    set: function reactiveSetter(newVal) {
      if (newVal === val) return
      val = newVal
      // Notify all dependencies
      dep.notify()
    }
  })
}
```

#### Why Dependency Collection is Needed
Let's look at an example. Here's a Vue object:
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
If we modify `text3`:
```javascript
data.text3 = 'text3 modify'
```
Although we modified the value of `text3` in `data`, since `text3` isn't used in the template rendering, it won't trigger `dep.notify()` to notify dependencies and update the view.

Here's another example. Suppose we have a global object that we use in two Vue objects.
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
When we modify the value of `globalObj.text1`:
```javascript
globalObj.text1 = 'text1 modify'
```
In the reactivity system, when `globalObj.text1` changes, it should notify both `o1` and `o2` vm instances to update their views. **Dependency collection** implements this: it lets this data (`globalObj.text1`) know that two places (`o1` and `o2`) depend on it, so when this data (`globalObj.text1`) changes, it will notify the `o1` and `o2` that depend on it.

#### Core of Dependency Collection - Dep and Watcher

Let's return to the code to see how dependency collection is implemented.

First, we need a `Dep`. Each `Dep` instance has an array to store watchers, which will notify watchers to update when data changes. Similarly, each `Watcher` instance also has an array to store its dependent deps. In other words, **dependency collection is bidirectional** - Dep records Watcher, and Watcher also records Dep, this is very important.

Let's look at `Dep` first:
```javascript
class Dep {
  static target  // Static property, globally unique currently executing Watcher
  id  // Instance property, unique identifier for each dep
  subs  // Instance property, stores all subscribers

  constructor {
    this.id = uid++
    this.subs = []
  }

  // Add subscriber
  addSub(sub) {
    this.subs.push(sub)
  }

  // Remove subscriber
  removeSub(sub) {
    remove(this.subs, sub)
  }

  // Establish dependency relationship
  depend() {
    if(Dep.target) { // If there is a currently executing Watcher, use it as dependency
      Dep.target.addDep(this)
    }
  }

  // Notify all subscribers
  notify() {
    for(let i = 0, l = this.subs.length; i < l; i++) {
      this.subs[i].update()
    }
  }
}

Dep.target = null;
```

As mentioned above, `class Dep`'s target is a static property used to record the currently unique executing Watcher, `id` is used to give Dep instances unique identifiers, and `subs` is used to store all subscribers. `addSub()` and `removeSub()` are used to add and remove subscribers, `depend()` is used to establish dependency relationships, and `notify()` is used to notify all subscribers.

`Watcher` is a bit more complex, let's take a look:

```javascript
class Watcher {
  constructor (vm, expOrFn, cb, options={}) {
    if(options) {
      this.lazy = !!this.lazy
      this.sync = !!this.sync
    }

    this.vm = vm  // Vue instance
    this.cb = cb  // Callback function
    this.deps = []  // Store all deps this watcher depends on
    this.newDeps = []  // New deps collected in this round
    this.depIds = new Set()  // Set of dependency IDs for deduplication
    this.newDepIds = new Set()  // Set of new dependency IDs
    this.expression = expOrFn.toString()  // For debugging
    this.dirty = this.lazy
    this.id = ++uid  // Unique identifier

    // Define getter here
    if(typeof expOrFn === 'function') {
      this.getter = expOrFn  // If expOrFn is a function, use it directly as getter
    } else {
      this.getter = parsePath(expOrFn) // If it's a string, convert to function that gets object property
    }

    // Execute getter immediately once for initialization and dependency collection
    this.value = this.lazy ? undefined : this.get()
  }

  get() {
    pushTarget(this)  // Set current watcher as globally active watcher
    let value
    try {
      // Execute getter, triggering observed object's getter, thus collecting dependencies
      value = this.getter.call(this.vm, this.vm)
    } catch (e) {
      throw e
    } finally {
      popTarget()  // Restore previous active watcher
      this.cleanupDeps()  // Clean up dependencies
    }
    return value
  }

  addDep(dep) {
    const id = dep.id
    // Check if already in new dependency collection
    if(!this.newDepIds.has(id)) {
      // Add to new dependency ID set
      this.newDepIds.add(id)
      // Add to new dependency array
      this.newDeps.push(dep)
      // Check if in old dependencies
      if(!this.depIds.has(id)) {
        // If not in old dependencies, let dep collect current watcher
        dep.addSub(this)
      }
    }
  }

  // Clean up dependencies
  cleanupDeps() {
    let i = this.deps.length
    // Traverse old dependency array
    while (i--) {
      const dep = this.deps[i]
      // If new dependency array doesn't include this old dependency
      if (!this.newDepIds.has(dep.id)) {
        // Remove this watcher from dep
        dep.removeSub(this)
      }
    }
    // Update depIds, clear newDepIds
    let tmp: any = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    // Update deps, clear newDeps
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  update() {
    // Lazy mode, lazy computation, won't recompute immediately, just mark dirty as true
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {  // Sync mode, synchronous update mode, update immediately
      this.run()
    } else {  // Default async update mode, put watcher into update queue
      queueWatcher(this)
    }
  }

  run() {
    const value = this.get()  // Get new value
    if(value !== this.value || isObject(value) || this.deep) {  // If new value != old value | new value is object | deep watching
      const oldValue = this.value  // Save old value
      this.value = value  // Set new value
      this.cb.call(this.vm, value, oldValue)  // Execute callback function, passing new and old values
    }
  }
}
```
In `Watcher`'s `constructor`, it first processes the passed `expOrFn`. When it's a function, it's directly assigned to `getter`. If it's a string, it needs `parsePath` to convert it to an access function. For example, `parsePath('user.name')` would return a function: `(obj) => obj['user']['name']`. Note that `obj` here is the `vm` called in `getter.call`, which is the current Vue instance. Additionally, it will immediately execute `getter()` once for initialization and dependency collection.

Looking at the code below, to handle nested Watcher scenarios, at the start of `getter()`, the current `Watcher` (target) is pushed onto the targetStack stack, and `Dep.target` is pointed to the current `Watcher`. When `getter()` finishes, this `Watcher` is popped from the stack, and `Dep.target`'s pointer is modified to the previous `Watcher`.

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

The `addDep` method of `Watcher` and the `addSub` method in `Dep` call each other for bidirectional dependency collection. Don't worry about infinite loops, as `Watcher` also has a `cleanupDeps` to clean up dependencies in a timely manner.

`update()` and `run()` are responsible for update handling when data changes. Let's first look at `update()`, which has three ways of handling when data changes:

1. `lazy` mode
   ```javascript
   if(this.lazy) {
    this.dirty = true
   }
   ```
   
   Searching for `dirty` in the source code, we can find the `evaluate()` method:
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

   When `evaluate()` is called, it calls the `get()` method and sets `dirty` to `false`. So where is `evaluate()` used?
   After a global search, we can see it being called in the `state.ts` file:

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

  In other words, when the `computed` property's `getter()` is called, if this watcher is dirty data (`dirty` is `true`), then `evaluate()` will be executed. Using `lazy` mode won't immediately recalculate values, it just marks `dirty` as `true`, waiting until the next access to this property to actually calculate.
  The computed property uses this mode.

2. `sync` mode
   ```javascript
   else if(this.sync) {
    this.run()
   }
   ```
   If in `sync` mode, it will call `run()`:
   ```javascript
   run() {
     const value = this.get()  // Get new value
     if(value !== this.value || isObject(value) || this.deep) {  // If new value != old value | new value is object | deep watching
       const oldValue = this.value  // Save old value
       this.value = value  // Set new value
       this.cb.call(this.vm, value, oldValue)  // Execute callback function, passing new and old values
     }
   }
   ```
   We can see that the `run` method immediately executes `get()`, and when the new and old values are not equal or the new value is an object or deep watching is enabled, it executes the callback function, passing both new and old values.
   So `sync` mode is mainly used for scenarios that need immediate response to data changes, but `sync` is `false` by default, and most cases use `async` asynchronous updates.

3. `queue` asynchronous updates
   ```javascript
   else {
    queueWatcher(this)
   }
   ```
   Let's see what `queueWatcher()` does:
   ```javascript
   let has = {}
   /**
     * Push a watcher into the watcher queue.
     * Jobs with duplicate IDs will be skipped unless it's
     * pushed when the queue is being flushed.
   */
   export function queueWatcher(watcher: Watcher) {
     // Duplicate check to prevent the same watcher from being added to the queue multiple times
     const id = watcher.id
     if (has[id] != null) {
       return
     }
     
     // Prevent watcher from triggering recursive updates during its own update process
     if (watcher === Dep.target && watcher.noRecurse) {
       return
     }
   
     has[id] = true
     if (!flushing) {  // If queue hasn't started flushing, push into queue
       queue.push(watcher)
     } else {
       // If queue is flushing, need to insert according to id order
       // Ensure watchers execute in order of creation (id size) because: 1.parent component watchers need to update before child components 2.computed needs to update before regular watchers
       let i = queue.length - 1
       while (i > index && queue[i].id > watcher.id) {
         i--
       }
       queue.splice(i + 1, 0, watcher)
     }
     // queue the flush
     if (!waiting) {
       waiting = true
       // Ensure queue will be flushed in next tick
       nextTick(flushSchedulerQueue)
     }
   }
   ```
   - When `flushSchedulerQueue` is executed, `flushing` will be set to `true`
   - By calling `flushSchedulerQueue()` through `nextTick()`, it executes `flushSchedulerQueue()` in the next **microtask**, so multiple data modifications will only trigger one update.

### Vue Reactivity System

Through the above sections, we know that Vue's reactivity system implementation relies on **data reactivity** and **dependency collection**. The core/principle of data reactivity is `Object.defineProperty`, which registers `get` and `set` for dependency collection. Specifically, it creates a new Dep instance in the reactive object to handle dependency collection, adding the current Watcher to the Dep instance's subscriber list (subs). At this point, the process of building the entire reactivity system is still somewhat unclear, especially when `new Watcher()` happens, so when we create a new Vue instance, when are all these processes performed?

First, we need to know what Watchers exist in a Vue instance. As Vue users, we commonly encounter three types: Render Watcher, Computed Watcher, and User Watcher:

```javascript
// Template rendering - Render Watcher
<div>{{ message }}</div>

// computed option - Computed Watcher
computed: {
  total() {
    return this.price * this.quantity
  }
}

// watch option - User Watcher
watch: {
  price(newVal, oldVal) {
    console.log('price change:', newVal)
  }
}
```

Now, let's not rush to focus on when `new Watcher` happens, let's first understand the overall flow of `new Vue()`:

```javascript {linenos=table, hl_lines=["14-15" "20-23"]}
function initMixin(Vue) {
  Vue.prototype._init = function () {
    const vm = this
    // Initialize lifecycle
    initLifeCycle(vm)
    // Initialize events
    initEvent(vm)
    // Initialize render
    initRender(vm)
    // Call beforeCreate Hook
    callHook(vm, 'beforeCreate')
    // Initialize inject
    initInject(vm)
    // Initialize state
    initState(vm)
    // Initialize provide
    initProvide(vm)
    // Call created Hook
    callHook(vm, 'created')
    // If there's an el option, mount automatically
    if(vm.$options.el) {
      vm.$mount(vm.$options.el)  // Create Render Watcher
    }
  }
}
```

We mainly focus on `initState` and `$mount`.

#### initState

`initState` initializes `props` -> `methods` -> `data` -> `computed` -> `watch` in sequence:

```javascript {linenos=table, hl_lines=["10-13" "15-16" "18-21"]}
function initState(vm) {
  const opts = vm.$options
  
  // 1. Initialize props
  if (opts.props) initProps(vm, opts.props)
  
  // 2. Initialize methods
  if (opts.methods) initMethods(vm, opts.methods)
  
  // 3. Initialize data
  if (opts.data) {
    initData(vm)  // Generate reactive object
  }
  
  // 4. Initialize computed
  if (opts.computed) initComputed(vm, opts.computed)
  
  // 5. Initialize watch
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```
The process of **generating reactive objects**, which we mentioned earlier using `Object.defineProperty`, is implemented in `initData`:

```javascript
function initData() {
  observe(data)  // Entry function for generating reactive objects mentioned earlier
}
```

`initComputed` and `initWatch` create Computed Watcher and User Watcher respectively:

```javascript
function initComputed() {
  // Create computed watcher for each computed property
  new Watcher(vm, getter, null, options)
}
function initWatch() {
  // Create user watcher for each watch
  new Watcher(vm, key, cb, options)
}
```

#### $mount

Render Watcher is created during component mounting (`$mount`).

```javascript
// Create render watcher (during mount phase)
Vue.prototype.$mount = new Watcher(vm, updateComponent, null, options)
```

## Vue2 Reactivity System Limitations

Vue2's reactivity isn't perfect. The implementation using `Object.defineProperty` comes with inherent limitations, so when using Vue2, we need to be aware of these issues.

### Objects

Vue2 cannot detect **object property additions and deletions**.

For example:

```javascript
var vm = new Vue({
 data: {
   a: 1
 }
})
// vm.a is reactive
vm.b = 2
// vm.b is non-reactive
```

This is because Vue converts properties to reactive ones during instance initialization, so properties must exist on the `data` object for Vue to make them reactive.

```javascript
function initData(vm) {
  let data = vm.$options.data
  observe(data)  // Call observe here to make object reactive
}
```

#### Solution 1: `Vue.set`

For already created instances, while Vue doesn't allow adding reactive properties at the **root level**, you can use `Vue.set(object, propertyName, value)` to add reactive properties to nested objects.

For example, if you have an object in data:

```javascript
var vm = new Vue({
  data: {
    someObject: {
      a: 1,
    }
  }
})
```
Use `Vue.set` to add a new reactive property `b` to `someObject`:

```javascript
Vue.set(vm.someObject, 'b', 2)
```
or

```javascript
this.$set(this.someObject, 'b', 2)
```

#### Solution 2: `Object.assign`

When you need to assign multiple new properties to an existing object, you can use `Object.assign`.

```javascript
this.someObject = Object.assign({}, this.someObject, {a: 1, b: 2})
```

However, note that new properties added like this aren't reactive:

```javascript
Object.assign(this.someObject, {a: 1, b: 2})
```

The reason is simple - `Object.assign(this.someObject,{a:1,b:2})` is equivalent to directly modifying/adding properties:

```javascript
// Equivalent to
this.someObject.a = 1  // Triggers a's setter
this.someObject.b = 2  // b remains non-reactive
```

While `Object.assign({},this.someObject,{a:1,b:2})` triggers `someObject`'s setter, recursively traversing all properties on `someObject` for reactive conversion.

### Arrays

Vue2 cannot detect changes to **array indices** and **length**.

- Array indices: `vm.items[indexOfItem] = newValue`
- Array length: `vm.items.length = newLength`

These two types of changes won't trigger reactive updates because array indices are essentially object properties. While theoretically Vue could handle array indices like object properties, for arrays with large lengths, like an array of 10000 elements, setting getter/setter for each index would cause serious **performance issues**. Similarly, the `length` property of arrays isn't made reactive because it could trigger chain reactions (modifying `length` could affect many elements).

To solve this problem, Vue2 rewrote seven array methods that can trigger reactive updates:

```javascript
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

So to trigger array reactivity, you should do this:

```javascript
// Modify array index
vm.items.splice(indexOfItem, 1, newValue)
// Modify array length
vm.items.splice(2)
```

### Performance

Vue2's reactive implementation requires **recursive traversal** of all object properties, which itself has significant performance overhead. This is also why Vue3 switched to using `proxy` to implement reactivity.

## Vue3's Improvements to Reactivity

### Implementing Reactivity with Proxy

Vue3's reactivity system is based on ES6's `proxy`, which is a major upgrade from Vue2's `Object.defineProperty`. The advantages include:
1. Can detect object property additions and deletions
2. Can monitor array changes without additional handling
3. No need for deep recursive traversal, better performance
4. Supports data structures like Map, Set

```javascript
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const res = Reflect.get(target, key, receiver)
      track(target, key)  // Dependency collection
      return res
    },
    set(target, key, value, receiver) {
      const res = Reflect.set(target, key, value, receiver)
      trigger(target, key)  // Trigger updates
      return res
    }
  })
}
```

Here `Reflect` is a built-in object that provides methods for intercepting JavaScript operations.

```javascript
Reflect.get(target, key)  // Get property
Reflect.set(target, key, value)  // Set property
Reflect.has(target, key)  // Check property
Reflect.deleteProperty(target, key)  // Delete property
```
Reflect provides unified APIs for object operations, making it convenient for proxy:

```javascript {linenos=table, hl_lines=["4-5" "10-11"]}
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      // Using Reflect ensures correct this binding
      const res = Reflect.get(target, key, receiver)
      track(target, key)
      return res
    },
    set(target, key, value, receiver) {
      // Returns boolean indicating if operation succeeded
      const res = Reflect.set(target, key, value, receiver)
      trigger(target, key)
      return res
    }
  })
}
```
### New Reactive APIs

#### ref and reactive

`ref` and `reactive` are both new reactive APIs in Vue3 for handling reactive data, with some differences in usage:

| Feature | ref | reactive |
|---------|-----|----------|
| Access method | Access via `.value` | Direct access |
| Auto-unwrapping | Auto-unwraps in `<template>` and `reactive` | No unwrapping needed |
| Data type support | Supports all data types | Only supports object types |
| Destructuring behavior | Loses reactivity after destructuring, need `toRef`/`toRefs` | Loses reactivity after destructuring, need `toRef`/`toRefs` |
| Assignment characteristic | Can directly replace entire value `ref.value = newValue` | Can't directly replace entire object, can only modify properties |
| Nested data | Internally uses `reactive` to handle objects | Deep reactive conversion |
| Use cases | Basic data types / Single data source / Composition function return values / Data needing reassignment | Related data collections / Reference data types / Data not needing reassignment |

- Access Method

`ref` objects are accessed via `.value`, `reactive` accessed directly.

```js
const count = ref(0)
console.log(count.value)  // needs .value to access

const obj = reactive({
  count: 0
})
console.log(obj.count)  // direct access, no .value needed
```

- Auto-unwrapping

`ref` objects auto-unwrap in `template`, `reactive`
```html
<template>
  <div>
    <!-- Already auto-unwrapped, direct access, no .value needed --->
    {{ count }}
  </div>
</template>

<script setup>
const count = ref(0)
// ref auto-unwraps in reactive objects
const state = reactive({
  count, // auto-unwrapped
  double: computed(() => state.count*2),
})
</script>
```

- Data Type Support
```js
// ref supports all data types
const num = ref(0)
const str = ref('XD')
const boo = ref(true)
const obj = ref({a:1, b:2})
const arr = ref([1,2])

// reactive only supports reference data types (objects/arrays)
const obj2 = reactive({a:1, b:2})
const arr2 = reactive([1,2])
```

- Using `toRef` / `toRefs` for Destructuring

```js
const obj = reactive({name: 'River', age: 18})

const { age } = obj  // Direct destructuring will lose reactivity

// Using toRef
const age = toRef(obj, 'age')
// Or toRefs
const { name, age } = toRefs(obj)

// The benefit is maintaining object reactivity
// Modifying ref will update source object
age.value++
console.log(obj.age) // 19
// Modifying source object will update ref
obj.age++
console.log(age.value) // 20
```
- Assignment Characteristics

`ref` objects can be directly replaced, `reactive` cannot be directly replaced, only properties can be modified
```js
const foo = ref([1,2])
foo.value = [3,4]  // allowed

const foo = reactive([1,2])
foo = [3,4]  // not allowed
```

- Nested Data

`ref` handling nested data
```js
const user = ref({
  name: 'Zhang',
  profile: {
    age: 25,
    address: {
      city: 'Beijing'
    }
  }
})

// ref internally uses reactive for deep object conversion
user.value.profile.age = 26      // triggers reactive update
user.value.profile.address.city = 'Shanghai' // triggers reactive update
```

`reactive` handling nested data
```js
const user = reactive({
  name: 'Zhang',
  profile: {
    age: 25,
    address: {
      city: 'Beijing'
    }
  }
})

// reactive deeply converts all nested objects
user.profile.age = 26           // triggers reactive update
user.profile.address.city = 'Shanghai' // triggers reactive update
```

- Use Cases
  
For composition function return values, using `ref` is better. If `reactive` is needed, maintain data reactivity through `toRefs`
```js
function useCount() {
  const count = ref(0)
  return count
}

function useUser() {
  const state = reactive({name: 'River', age: 18})
  return toRefs(state)
}
```

#### watchEffect

`watchEffect` automatically tracks reactive dependencies and reruns the effect function when reactive dependencies update. Simply put, it does these things:
- Immediately executes the callback function once
- Automatically tracks reactive dependencies used in the callback function
- Reruns the callback function when dependencies change

##### Basic Usage
```js
import { ref, watchEffect } from 'vue'

const count = ref(0)
const message = ref('Hello')

watchEffect(() => {
  console.log(`Count: ${count.value}, Message: ${message.value}`)
})

// Modifying any dependency will trigger callback
count.value++  // Output: Count: 1, Message: Hello
message.value = 'Hi'  // Output: Count: 1, Message: Hi
```

##### Pause/Resume/Stop Watching

`watchEffect` also returns a stop function, executing it will stop watching

```js
const stop = watchEffect(() => {})

// When watching is no longer needed
stop()
```

When pausing/resuming is needed
```js
const { stop, pause, resume } = watchEffect(() => {})

// Pause
pause()

// Resume
resume()

// Stop
stop()
```

##### Cleanup Effects

Why cleanup effects are needed:
- Prevent memory leaks (like timers)
- Avoid duplicate event listeners
- Cancel unnecessary network requests
- Clean up potentially conflicting old states

The `onCleanup` parameter in `watchEffect`'s callback function is used to clean up side effects. It executes at these times:
- Just before `watchEffect` is about to re-execute
- When `watchEffect` is stopped

Let's take a network request as an example, creating a network request controller `controller` and calling the cancel network request method in the `onCleanup` function.

```js
const userId = ref('1')
const userData = ref(null)

watchEffect((onCleanup) => {
  // Create a cancel controller
  const controller = new AbortController()

  // Network request
  fetch(`/api/user/${userId.value}`, {
    signal: controller.signal,
  }).then(data => userData.value = JSON.parse(data))

  // Cleanup function: cancel previous request if userId changes
  onCleanup(() => {
    controller.abort()
  })
})

setTimeout(() => {
  userId.value = '2'
}, 100)
```

The execution sequence is like this:
```markdown {filename=Markdown}
Initial execution:
1. Start request, userId: 1

100 ms later when userId changes:
2. Execute cleanup, cancel previous network request (cleanup previous side effect `onCleanup`)
3. Request canceled (previous request `abort`)
4. Start request, userId: 2 (re-execute `watchEffect`)
```

Cleanup after 3.5+
```js
import { onWatcherCleanup } from 'vue'

// ...

watchEffect(() => {
  // ...

  onWatcherCleanup(() => {
    controller.abort()
  })

  // ...
})

// ...
```

##### Execution Timing

`watchEffect` also provides a second parameter that can control when the effect function executes.

```js {linenos=table, hl_lines=["7" "12"]}
// Default: execute before component updates, flush: 'pre'
watchEffect(() => {})

// Execute after component updates
watchEffect(() => {
  // ...
}, { flush: 'post' })

// Synchronous execution
watchEffect(() => {
  // ...
}, { flush: 'sync' })
```

##### watchEffect vs watch

- `watchEffect` automatically tracks dependencies, `watch` needs explicitly specified source to monitor
```js
watchEffect(() => console.log(count.value))  // automatically tracks dependencies

watch(count, newVal => console.log(count.value))  // explicitly specifies dependency
```

- `watchEffect` executes immediately by default, `watch` needs `immediate: true` setting
```js
watchEffect(() => {})  // executes immediately by default

watch(source, () => {}, { immediate: true })  // won't execute immediately by default, needs immediate set to true
```
