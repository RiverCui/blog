---
title: 'Node.js 前置知识'
date: 2023-11-06
draft: false
description: "Node.js 学习笔记"
summary: "Node.js 并不是一种语言而是一个平台，是 JavaScript 的**运行时环境**。"
tags: ["Node.js"]
series: ["Node.js"]
series_order: 1
---
## Node.js 是 JavaScript 运行时环境

> Node.js® is an open-source, cross-platform JavaScript runtime environment.

引用来源：[Node.js](https://nodejs.org/en)

Node.js 并不是一种语言而是一个平台，是 JavaScript 的**运行时环境**(runtime environment)/宿主环境(host environment)。

讨论 Node.js 就不得不提到 Chrome V8 引擎。09年，谷歌开始研发 Chrome 浏览器，这也是现今使用最为广泛的浏览器，由 Lars Bak 领导开发的 Chrome V8 引擎也相应问世，JavaScript 就此展开了一场性能革命，Node.js 就是基于 Chrome V8 引擎构建的。随后，Node.js 的作者 Ryan Dahl 还构建了 [Deno](https://github.com/denoland/deno)。

如今，JavaScript 的层级架构也基本趋于稳定，如图：

![JavaScript 架构图](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/js-architecture-diagram.png)
图片来源：[掘金小册](https://juejin.cn/book/7196627546253819916/section/7195089399787290635)

- 最下面的层级是**脚本语言规范**(Spec)，[ECMAScript](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/)。
- 再往上一层是语言实现，JavaScript、JScript、ActionScript 等都是对 ECMAScript 的语言实现。
- 到了引擎层面，除了上文提到的 Chrome V8 外，还有 SpiderMonkey、QuickJS、JerryScript 等常见引擎。
- 最上面是运行时环境，基于 Chrome V8 封装的运行时环境有 Node.js、Chromium、Deno、CloudFlare Workers，Firefox 是基于 SpiderMonkey 封装的运行时环境。

## 基本架构

![Node.js 架构图](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/nodejs-architecture-diagram.png)
图片来源：[medium.com](https://chathuranga94.medium.com/nodejs-architecture-concurrency-model-f71da5f53d1d)

Node.js 是运行在操作系统之上的，底层由 Chrome V8 引擎和一些 C/C++ 写的库构成，比如 libUv、c-ares、llhttp/http-parser、open-ssl、zlib等。其中，libUV 负责处理事件循环，c-ares、llhttp/http-parser、open-ssl、zlib 等库提供 DNS 解析、HTTP 协议、HTTPS 和文件压缩等功能。

中间层由 Node.js Bindings、Node.js Standard Library 和 C/C++ AddOns 构成。**Node.js Bindings** 层的作用是将底层那些用 C/C++ 写的库接口暴露给 JS 环境，而 **Node.js Standard Library** 是 Node.js 本身的核心模块。至于 **C/C++ AddOns**，它可以让用户自己的 C/C++ 模块通过桥接的方式提供给Node.js。

再上一层是 Node.js 的 API 层，我们使用 Node.js 开发应用，主要使用的就是 API 层，Node.js 应用最终运行在 API 层之上。

## 使用场景

- 服务端
  
  Node.js 提供了基于事件驱动和非阻塞的接口，可用于编写高并发状态下的程序，而且 JavaScript 的匿名函数、闭包、回调函数等特性就是为事件驱动而设计的。

  Node.js 一些内置模块如 `http`、`net`、`fs`，就是为服务端设计的。无论是 HTTP，还是 TCP、UDP，又或是 RPC 服务，Node.js 都可以胜任。

- 桌面端
  
  [Electron.js](https://www.electronjs.org/)，由OpenJS基金会开发和维护的自由开源软件框架。该框架旨在使用Chromium浏览器引擎版本和使用Node.js运行时环境的后端渲染的网络技术创建桌面应用程序。

  Visual Studio Code、1Password、钉钉等软件，都是基于 Electron 开发的。
  
- 端游
  
  除了通过 WebGL 在浏览器里写前端页面上的游戏、通过 Electron 封装成看起来像端游的游戏外，JavaScript 还能通过 Node.js 的 binding 去使用 OpenGL 甚至 DirectX 去写货真价实的桌面游戏。

- 机器学习
  
  [pipcook](https://github.com/alibaba/pipcook)

Node.js 最早被设想为一种用于构建服务端应用的技术，但作为运行在操作系统中的 JavaScript 运行时环境，它已经成为了一个平台，可适配领域涵盖了泛前端和后端，传统服务和 Serverless，工具、商业、游戏等等。
