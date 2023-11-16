---
title: '模块管理'
date: 2023-11-06
draft: false
description: "Node.js 学习笔记"
summary: "`CommonJS` 和 `ES Modules` 是 JavaScript 中两种主要的模块系统。"
tags: ["Node.js"]
series: ["Node.js"]
series_order: 2
---
## 模块化及其优势

Node.js 与 浏览器的 JavaScript 最大不同就在于 **Node.js 是模块化的**。

### 模块化

模块化是一种编程范式，将大型、复杂的程序系统分解成更小、更易管理和维护的部分。在模块化中，每个模块执行一项特定的功能，同时尽可能减少与其他模块的直接交互。这样的方法有很多优点：

1. **封装**。每个模块将数据和功能封装在模块内部，并提供接口与外界交互。有助于隐藏内部实现细节，减少模块间的相互依赖。
2. **重用性**。模块化可以使开发者在项目多处重复使用代码块，也可以在多个项目或应用程序中使用，减少重复代码的编写，提高代码整体质量。
3. 可维护性和可读性。模块化代码通常更易于理解和维护，每个模块负责清晰定义的功能，使得代码更加直观和易于管理。
4. **独立性**。模块之间的松耦合确保了修改一个模块不会或很少影响其他模块，有助于添加、更新和修复功能。

### Node.js 的模块化选择

在 Node.js 诞生之初，JavaScript 还没有标准的模块机制，因此 Node.js 一开始采用了 [`CommonJS`](https://en.wikipedia.org/wiki/CommonJS)(下文简写为 `CJS`)。随后，JavaScript 标准的模块化机制 `ES Modules`(下文简写为 `ESM`) 诞生，浏览器开始逐步支持 `ESM`。在 Node.js 支持 `ESM` 之前，就有 `Babel` 这样的编译工具和 `Webpack` 这样的打包工具，将规范的 `ESM` 模块机制编译成 Node.js 的 `CJS` 模块机制了。随后，在Node.js `v13.2.0` 也开始引入了规范的 `ESM` 机制，同时兼容早期 `CJS`。

所以我们现在写 Node.js 模块的时候，有 3 种思路：

1. 直接采用 `ESM`，在 Node.js `v13.2.0` 之后的版本可行。
2. 使用 `ESM`，但是通过 `Babel` 编译成 `CJS`。
3. 使用 `CJS`，Node.js 在未来很长一段时间还是会同时兼容 `ESM` 和 `CJS` 的。


## `ES Modules`
**`export` 导出， `import` 导入**

### 导出语法
- exporting declaration 导出声明
  ```javascript
  export let a, b
  export const a = 1, b = 2
  export function functionName () {}
  export class ClassName {}
  export const { a, b } = obj
  export const [ a, b ] = arr
  ```
  例如：
  ```javascript
  // hello.mjs
  export const name = 'River'

  // index.mjs
  import { name } from './hello.mjs'
  ```

- export list 导出列表
  ```javascript
  export { name1, name2 }
  export { variable1 as name1, variable2 as name2 }
  export { variable1 as 'string name' }
  export { name1 as default }
  ```
  例如：
  ```javascript
  // hello.mjs
  const name = 'River'
  const sayHello = (text) => `Hello ${text}!`
  export { name, sayHello as default }

  // index.mjs
  import { name } from './hello.mjs'
  import sayHello from './hello.mjs'
  console.log(sayHello(name)) // Hello River!
  ```
- default exports 默认导出
  ```javascript
  export default expression
  export default function functionName() {}
  export default class ClassName {}
  ```

此外，还有 `export ... from ...`的聚合语法，在此不做赘述，要想了解可以看 [MDN-JavaScript-export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#re-exporting_aggregating)。

### 导入语法
基于上述的 `export` 导出方式可以知道，`export` 导出的 API 分为 default 和 非 default。总结一下，针对这两种 API，`import` 也有两种不同的导入方式（可查看上述例子）：
```javascript
import { API } from 模块路径 // 非 default API
import defaultAPI from 模块路径 // default API
```
当然，和 `export` 一样，`import` 也可以使用 `as` 给 API 重命名。
```javascript
import { variable1 as name1 } from 模块路径
import * as foo from 模块路径
```
上述的 `import * as foo` 可以将导出 API 生成对象 `foo`，并且 **default API 为 foo.default**。

### 文件名后缀
需要注意的是，在 Node.js 中默认使用 `CommonJS` 规范定义 `.js` 文件的模块，用 `ES Modules` 规范定义 `.mjs` 文件的模块。关于这两者的启用规则，详细可查看 [nodejs doc](https://nodejs.org/docs/latest/api/modules.html#enabling)，在此不做赘述。

如果要使用 `ESM` 定义 `.js` 文件的模块，可以在配置文件 `package.json` 中设置 `type: module`。

## `CommonJS`
`module.exports` 导出，`require` 导入

### 导出语法
- `module.exports`
  ```javascript
  module.exports = { name1, name2 }
  module.exports = { name1: variable1, name2: variable2 } // 不同于 ES Module 的 as 用法
  ```
- `exports`
  ```javascript
  exports.a = 1
  exports.b = 2
  exports.functionName = () => a + b
  ```

例如：
```javascript
// hello.js
const name = 'River'
const sayHello = (text) => `Hello ${text}!`
module.exports = {
  name, sayHello
}

// index.js
const { name, sayHello } = require('./hello.js')
console.log(sayHello(name))
```
`exports.属性名` 是早期用法，现在应该尽量使用 `module.exports = { 属性名 }`。注意，**两种语法不能同时使用**，因为 `exports.属性名` 会被 `module.exports = { 属性名 }` 覆盖。

### 导入语法
```javascript
const { API } = require(模块路径)
```

## `ESM` 和 `CJS` 的差异

加载机制是理解这两个模块系统核心特性的关键，也是它们根本的区别所在。

### `CJS` 动态加载
- **运行时加载**。模块依赖关系在代码执行时解析。这意味着 `require()` 函数调用是在代码执行过程中处理的，可以灵活使用模版字符串动态拼接路径，例如：
  ```javascript
  const libPath = ENV.supportES6 ? './es6/' : './'
  const myLib = require(`${libPath}lib.js`)
  ```
  还可以根据程序的运行逻辑和条件来加载模块。例如，可以在 `if` 中调用 `require()`，根据不同的条件加载不同的模块。
  ```javascript
  let api;
  if(condition) {
    api = require('./foo');
  } else {
    api = require('./bar');
  }
  ```
- **同步加载**。在不需要考虑网络延迟的情况下，尤其是服务器端 JavaScript(Node.js)，模块一般都是从本地文件系统中加载，此时同步加载是可行的。

### `ESM` 静态加载
- **编译时加载**。模块依赖关系在编译时就确定。`import` 和 `export` 都必须放在最外层，不能被包含在函数或条件语句内。 
- **异步加载**。异步加载允许代码在模块下载和处理期间继续执行，解决了阻塞问题，当然也使模块管理更加复杂。

### `ESM` 实现动态加载
`ESM` 不允许 `import` 语句用动态路径，也不允许在语句块中使用，但允许通过使用 `import()` 函数实现**动态加载**，而且是**异步**进行的。

动态的 `import()` 返回一个 `Promise`，该 `Promise` 解析为引入模块的所有导出。需要使用 **`.then()`**、**`async/await`** 等方式来处理导入的模块。例如：
```javascript
(async function() {
  const { functionName } = await import('./myModules.mjs)
  functionName()
}())
```

使用场景：
1. 条件加载
  例如：
  ```javascript
  if(someCondition) {
    import('./myModules.mjs')
      .then(module => {
        module.functionName()
      })
  }
  ```
2. 性能优化。比如在处理大型模块时，通过代码分割和懒加载，减少应用的初始负载时间，提升性能。

## `ESM` 向下兼容
在 Node.js 环境中，`ES Modules` 会向下兼容 `CommonJS`，对于 `CJS` 导出的 API，也可以使用 `import()` 引入，并且只能以 **`default` 方式**引入。例如：
```javascript
// foo.js
const a = 1
const b = 2
const c = () => a + b
module.exports = { a, b, c }

// bar.mjs
import abc from './foo.js'
console.log(abc.a, abc.b, abc.c()) // 1, 2, 3
```

也就是说，`module.exports` 相当于：
```javascript
const abc = { a, b, c }
export default abc
```
