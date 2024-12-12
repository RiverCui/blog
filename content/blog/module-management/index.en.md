---
title: 'Module Management'
date: 2023-11-16
description: "Two main module systems in JavaScript: `CommonJS` and `ES Modules`"
tags: ["Node.js"]
series: ["Node.js"]
series_order: 2
---

> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

## Modularization and Its Benefits

The biggest difference between Node.js and browser JavaScript is that **Node.js is modular**.

### Modularization

Modularization is a programming paradigm that breaks down large, complex program systems into smaller, more manageable and maintainable parts. In modularization, each module performs a specific function while minimizing direct interaction with other modules. This approach has many advantages:

1. **Encapsulation**. Each module encapsulates data and functionality internally and provides interfaces for external interaction. This helps hide internal implementation details and reduces interdependencies between modules.
2. **Reusability**. Modularization allows developers to reuse code blocks across multiple parts of a project, or across multiple projects or applications, reducing duplicate code and improving overall code quality.
3. **Maintainability and Readability**. Modular code is typically easier to understand and maintain, with each module responsible for clearly defined functionality, making the code more intuitive and easier to manage.
4. **Independence**. Loose coupling between modules ensures that modifying one module will not or minimally affect other modules, facilitating the addition, updating, and fixing of functionality.

### Node.js's Module Choices

When Node.js was first created, JavaScript didn't have a standard module mechanism, so Node.js initially adopted [`CommonJS`](https://en.wikipedia.org/wiki/CommonJS) (abbreviated as `CJS` below). Later, when JavaScript's standard module mechanism `ES Modules` (abbreviated as `ESM` below) was born, browsers gradually began supporting `ESM`. Before Node.js supported `ESM`, compilation tools like `Babel` and bundling tools like `Webpack` were already compiling standard `ESM` module mechanisms into Node.js's `CJS` module mechanism. Subsequently, Node.js `v13.2.0` also introduced the standard `ESM` mechanism while maintaining compatibility with early `CJS`.

So now when writing Node.js modules, we have 3 approaches:

1. Directly use `ESM`, feasible in Node.js versions after `v13.2.0`.
2. Use `ESM` but compile to `CJS` through `Babel`.
3. Use `CJS`, as Node.js will continue to support both `ESM` and `CJS` for a long time to come.

## `ES Modules`
**`export` for exporting, `import` for importing**

### Export Syntax
- exporting declaration
  ```javascript
  export let a, b
  export const a = 1, b = 2
  export function functionName () {}
  export class ClassName {}
  export const { a, b } = obj
  export const [ a, b ] = arr
  ```
  For example:
  ```javascript
  // hello.mjs
  export const name = 'River'

  // index.mjs
  import { name } from './hello.mjs'
  ```

- export list
  ```javascript
  export { name1, name2 }
  export { variable1 as name1, variable2 as name2 }
  export { variable1 as 'string name' }
  export { name1 as default }
  ```
  For example:
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
- default exports
  ```javascript
  export default expression
  export default function functionName() {}
  export default class ClassName {}
  ```

Additionally, there's the aggregating syntax `export ... from ...`, which won't be detailed here. For more information, see [MDN-JavaScript-export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#re-exporting_aggregating).

### Import Syntax
Based on the above `export` methods, we can see that exported APIs are either default or non-default. To summarize, there are two different import methods for these two types of APIs (see examples above):
```javascript
import { API } from module-path // non-default API
import defaultAPI from module-path // default API
```
Of course, like `export`, `import` can also use `as` to rename APIs.
```javascript
import { variable1 as name1 } from module-path
import * as foo from module-path
```
The above `import * as foo` can generate an object `foo` from exported APIs, and **the default API becomes foo.default**.

### File Extensions
Note that in Node.js, `.js` files use the `CommonJS` specification by default for defining modules, while `.mjs` files use the `ES Modules` specification. For detailed rules about enabling these two, see the [nodejs doc](https://nodejs.org/docs/latest/api/modules.html#enabling).

To use `ESM` to define modules in `.js` files, you can set `type: module` in the `package.json` configuration file.

## `CommonJS`
`module.exports` for exporting, `require` for importing

### Export Syntax
- `module.exports`
  ```javascript
  module.exports = { name1, name2 }
  module.exports = { name1: variable1, name2: variable2 } // Different from ES Module's as usage
  ```
- `exports`
  ```javascript
  exports.a = 1
  exports.b = 2
  exports.functionName = () => a + b
  ```

For example:
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
`exports.propertyName` is an early usage; now we should preferably use `module.exports = { propertyName }`. Note that **these two syntaxes cannot be used simultaneously**, as `exports.propertyName` will be overwritten by `module.exports = { propertyName }`.

### Import Syntax
```javascript
const { API } = require(module-path)
```

## Differences Between `ESM` and `CJS`

The loading mechanism is key to understanding the core features of these two module systems and represents their fundamental difference.

### `CJS` Dynamic Loading
- **Runtime Loading**. Module dependencies are resolved during code execution. This means `require()` function calls are processed during code execution, allowing flexible use of template strings for dynamic path concatenation, for example:
  ```javascript
  const libPath = ENV.supportES6 ? './es6/' : './'
  const myLib = require(`${libPath}lib.js`)
  ```
  You can also load modules based on program logic and conditions. For example, you can call `require()` within an `if` statement to load different modules based on different conditions.
  ```javascript
  let api;
  if(condition) {
    api = require('./foo');
  } else {
    api = require('./bar');
  }
  ```
- **Synchronous Loading**. When network delay isn't a concern, especially in server-side JavaScript (Node.js), modules are generally loaded from the local file system, making synchronous loading feasible.

### `ESM` Static Loading
- **Compile-time Loading**. Module dependencies are determined at compile time. `import` and `export` must be placed at the top level and cannot be included in functions or conditional statements.
- **Asynchronous Loading**. Asynchronous loading allows code to continue executing while modules are downloading and processing, solving blocking issues but making module management more complex.

### Dynamic Loading with `ESM`
While `ESM` doesn't allow `import` statements with dynamic paths or within statement blocks, it does allow **dynamic loading** through the `import()` function, which is **asynchronous**.

Dynamic `import()` returns a `Promise` that resolves to all exports of the imported module. You need to use **`.then()`**, **`async/await`** or similar methods to handle the imported module. For example:
```javascript
(async function() {
  const { functionName } = await import('./myModules.mjs)
  functionName()
}())
```

Use cases:
1. Conditional Loading
  For example:
  ```javascript
  if(someCondition) {
    import('./myModules.mjs')
      .then(module => {
        module.functionName()
      })
  }
  ```
2. Performance Optimization. For example, when handling large modules, using code splitting and lazy loading to reduce initial load time and improve performance.

## `ESM` Backward Compatibility
In the Node.js environment, `ES Modules` is backward compatible with `CommonJS`. APIs exported by `CJS` can also be imported using `import()`, but only as a **`default` import**. For example:
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

In other words, `module.exports` is equivalent to:
```javascript
const abc = { a, b, c }
export default abc
```