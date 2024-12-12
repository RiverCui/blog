---
title: 'NPM'
date: 2023-11-17
description: "npm is the package manager for JavaScript."
tags: ["Node.js"]
series: ["Node.js"]
series_order: 3
---

> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

## NPM
NPM is the package manager that comes with Node.js.

Node.js modules are broadly divided into internal modules and other modules. Internal modules are integrated within Node.js and don't require referencing external JS files - they can be imported using `require` or `import` with the module name.

For example, the built-in fs module (for file operations) can be imported like this:

```javascript
const fs = require('fs')
```

![Node.js built-in modules](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/iShot_2023-11-16_16.17.43.png)

Besides these built-in modules shown in the image, we often use many other modules in development. There are two ways to import them:

- Import using file paths
  
  For example, in the project below, to import `foo.js` from directory `a` into `bar.js` in directory `b`:
  ```
  - a
    foo.js
  - b
    bar.js
  ```
  You can import it in `bar.js` like this:
  ```javascript
  const foo = require('../a/foo.js')
  ```
- Install modules into your project using NPM (by default in the node_modules directory) and reference them by package name in your code.
  
  [npm registry](https://www.npmjs.com/) is NPM's official repository where you can find needed packages and follow package documentation for installation and usage.

  For example, the [dayjs](https://www.npmjs.com/package/dayjs) plugin can be installed using NPM:
  ```javascript
  npm install dayjs
  ```

  Usage:
  ```javascript
  const dayjs = require('dayjs')
  dayjs().format()
  ```
  **When importing modules by package name, Node.js uses the `resolve` algorithm to first search the node_modules directory in the module's location. If not found, it recursively searches node_modules in parent directories up to the root directory.**

There's a lot worth discussing about NPM - I'll create a separate topic for it in the future ✍️.