+++
title = 'title'
date = 2024-11-27T14:36:46+08:00
description = ""
tags = ["Vue"]
draft = true
+++

# Vue 内部运行机制
本文梳理 Vue 的内部运行机制，重点着眼于响应式系统。

# Vue 的编译模式：运行时 + 编译时
首先了解下什么是Vue的运行时和编译时：
- 运行时：直接用 render 函数创建虚拟 DOM
- 编译时：需要把 template 编译成 render 函数

实际使用场景：
```vue
// 运行时
new Vue({
  render: h => h('div', 'Hello')
})

// 编译时
new Vue({
  template: `<div>Hello</div>`
})
```

编译时：
- 处理 Vue 文件
- 处理 template 选项
- 开发时的热更新
运行时：
- 编译后的代码运行
- 渲染流程
- 响应式系统

# 响应式系统的基本原理