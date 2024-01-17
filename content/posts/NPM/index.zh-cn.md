---
title: 'NPM'
date: 2023-11-17
draft: false
description: "Node.js 学习笔记"
summary: "npm 是 JavaScript 的包管理工具。"
tags: ["Node.js"]
series: ["Node.js"]
series_order: 3
---

## NPM 
NPM 是 Node.js 自带的包管理工具。

Node.js 的模块大致分为内部模块和其他模块。内部模块是 Node.js 内部集成的模块，不需要引用 JS 外部文件，而是通过 `require` 或 `import` 模块名引入。

比如内置 fs 模块（文件操作相关的模块），可以这样引入：

```javascript
const fs = require('fs')
```

![Node.js 内置模块](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/iShot_2023-11-16_16.17.43.png)

除了上图这些内置模块外，在开发中还会使用很多其他模块，有两种方式可以引入：

- 通过文件路径引入
  
  例如，在下面这个项目中，想要在 `b` 目录下的 `bar.js` 中引入 `a` 目录下的 `foo.js`。
  ```
  - a
    foo.js
  - b
    bar.js
  ```
  可以在 `bar.js` 里这样引入：
  ```javascript
  const foo = require('../a/foo.js')
  ```
- 通过 NPM 将模块安装到项目中(默认是 node_modules 目录)，在代码中用包名来引用模块。
  
  [npm registry](https://www.npmjs.com/) 是 NPM 的官方仓库，可查找需要的包，并按照包文档的指引进行安装使用。

  比如，[dayjs](https://www.npmjs.com/package/dayjs) 这个插件，使用 NPM 安装：
  ```javascript
  npm install dayjs
  ```

  使用：
  ```javascript
  const dayjs = require('dayjs')
  dayjs().format()
  ```
  **当通过包名引入模块时，Node.js 会根据 `resolve` 算法先搜索模块所在的目录下的 node_modules，如果没找到，会递归查找上级目录中的 node_modules，直到根目录。**

NPM 值得讲的很多，以后会单独开个专题✍️。