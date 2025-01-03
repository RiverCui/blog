---
title: 'Tree Shaking 的原理和使用方法'
date: 2024-12-20
description: "Tree Shaking 依赖于 ES6 Module 的静态特性，通过分析代码来剔除实际没有被使用的代码，从而实现前端构建优化。"
tags: ["Vue"]
---

## 是什么
Tree Shaking 是 JS 的一项重要**优化**技术，通过分析代码，删掉那些实际没有被使用的代码，减小生产构建的体积，从而提升应用的加载速度和性能。

## 原理

Tree Shaking 依赖于 **ES6 Module 语法的[静态特性](https://exploringjs.com/es6/ch_modules.html#static-module-structure)**，例如 `import` 和 `export`，通过静态分析来确定哪些模块和变量没有被使用，通常通过抽象语法树(AST)来实现。

Tree Shaking 的概念最早由 Rollup 普及，现在已经成为 JS 优化的共识。对于打包工具(Webpack、Rollup等)，在编译阶段，构建工具(Webpack、Rollup 等)会**分析并标记**未被引用的模块和变量，在最终打包时将其**剔除**。Vue3 也引入了**模块化设计**，允许按需导入，从而实现更好的 Tree Shaking。

## 构建工具配置 Tree Shaking
Webpack、Vite 和 Rollup 都支持 Tree Shaking，下面简单讲下如何配置以及注意事项。

### Webpack
1. 确保使用 ES6 模块语法，例如：
```javascript
export function add(a, b) {
  return a + b
}
export function subtract(a, b) {
  return a - b
}
```
2. 配置 `package.json`

在 `package.json` 中配置 `sideEffects`，告诉 Webpack 哪些文件有**副作用**，哪些文件可以安全的进行 Tree Shaking。


> [!TIP] 什么是 `sideEffects`?
> `sideEffects` 即**副作用**，指模块在被导入时，除了导出模块，还会执行一些额外的操作，例如：
> - 修改全局变量
> - 动态加载资源（如CSS文件）
> - 执行一些初始化逻辑

如果没有副作用文件，直接设置为 false：

```json
{
  "sideEffects": false
}
```

如果有副作用文件，可指定文件路径：

```json
{
  "sideEffects": [
    "*.css",
    "src/some-module.js"
  ]
}
```

3. 启用生产模式

在生产模式下，Webpack 会自动进行 Tree Shaking 和代码压缩

```javascript
module.exports = {
  mode: "production",  // 开启生产模式
  optimization: {
    useExports: true  // 开启 Tree Shaking
  }
}
```
> [!WARNING]
> 如果使用了 [Babel](https://github.com/babel/babel)，要确保它不会将 ES6 Module 转换为 CommonJS Module，在 `babelrc` 或 `babel.config.js` 中配置：
> ```js
> {
>   "presets": [
>     ["@babel/preset-env", { "modules": false }] // 保留 ES6 模块语法
>   ]
> }
> ```

### Vite 和 Rollup

Vite 和 Rollup 默认启用 Tree Shaking，确保使用 ES6 Module 语法，并在 `package.json` 文件中设置 `sideEffects: false`，打包即可。

## CSS Tree Shaking

除了 JS 外，通过 [purifyCSS](https://github.com/purifycss/purifycss) plugin，CSS 也可以实现 Tree Shaking。

https://cloud.tencent.com/developer/article/1617110

