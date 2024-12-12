---
title: 'Node.js Prerequisites'
date: 2023-11-06
description: "Node.js is not a language but a platform, serving as a runtime environment for JavaScript."
tags: ["Node.js"]
series: ["Node.js"]
series_order: 1
---

> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

## Node.js is a JavaScript Runtime Environment

> Node.js® is an open-source, cross-platform JavaScript runtime environment.<center><font size=2>Source: [Node.js](https://nodejs.org/en)</font></center>

Node.js is not a language but a platform, serving as a JavaScript runtime environment/host environment.

Any discussion of Node.js inevitably involves the Chrome V8 engine. In 2009, Google began developing the Chrome browser, now the most widely used browser, and the Chrome V8 engine developed under Lars Bak's leadership was born, launching a performance revolution for JavaScript. Node.js was built on the Chrome V8 engine. Later, Node.js creator Ryan Dahl also built [Deno](https://github.com/denoland/deno).

Today, JavaScript's architectural hierarchy has largely stabilized, as shown:

![JavaScript architecture diagram](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/js-architecture-diagram.png)<center><font size=2>Source: [Juejin Books](https://juejin.cn/book/7196627546253819916/section/7195089399787290635)</font></center>

- The bottom layer is the scripting language specification (Spec), [ECMAScript](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/).
- Above that is language implementation, with JavaScript, JScript, ActionScript, etc., all implementing ECMAScript.
- At the engine level, besides Chrome V8 mentioned above, there are SpiderMonkey, QuickJS, JerryScript, and other common engines.
- The top layer consists of runtime environments, with Node.js, Chromium, Deno, and CloudFlare Workers based on Chrome V8, while Firefox is based on SpiderMonkey.

## Basic Architecture

![Node.js architecture diagram](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/nodejs-architecture-diagram.png)
Source: [medium.com](https://chathuranga94.medium.com/nodejs-architecture-concurrency-model-f71da5f53d1d)

Node.js runs on top of the operating system, with its lower layer consisting of the Chrome V8 engine and some C/C++ libraries like libUv, c-ares, llhttp/http-parser, open-ssl, zlib, etc. LibUV handles event loops, while c-ares, llhttp/http-parser, open-ssl, zlib, and other libraries provide DNS resolution, HTTP protocol, HTTPS, and file compression capabilities.

The middle layer consists of Node.js Bindings, Node.js Standard Library, and C/C++ AddOns. The **Node.js Bindings** layer exposes the C/C++ library interfaces to the JS environment, while the **Node.js Standard Library** contains Node.js's core modules. As for **C/C++ AddOns**, they allow users to bridge their own C/C++ modules to Node.js.

Above is Node.js's API layer, which is what we primarily use when developing Node.js applications. Node.js applications ultimately run on top of the API layer.

## Use Cases

- Server-side
  
  Node.js provides event-driven and non-blocking interfaces for writing programs under high concurrency conditions. JavaScript's anonymous functions, closures, callbacks, and other features were designed for event-driven programming.

  Some of Node.js's built-in modules like `http`, `net`, `fs` were designed for server-side use. Whether it's HTTP, TCP, UDP, or RPC services, Node.js can handle them all.

- Desktop Applications
  
  [Electron.js](https://www.electronjs.org/), an open-source framework developed and maintained by the OpenJS Foundation, aims to create desktop applications using web technologies with the Chromium browser engine and Node.js runtime environment.

  Software like Visual Studio Code, 1Password, DingTalk are all developed using Electron.

- PC Games
  
  Besides writing browser-based games using WebGL or packaging games through Electron, JavaScript can also create genuine desktop games through Node.js bindings to use OpenGL or even DirectX.

- Machine Learning
  
  [pipcook](https://github.com/alibaba/pipcook)

While Node.js was originally conceived as a technology for building server-side applications, as a JavaScript runtime environment operating within the operating system, it has evolved into a platform spanning front-end and back-end development, traditional services and Serverless, tools, business applications, games, and more.