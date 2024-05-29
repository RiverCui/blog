---
title: '使用 mkcert/Vite 生成本地证书'
date: 2023-12-25
description: "介绍两种在本地运行 HTTPS 协议的方法：mkcert 和 Vite"
summary: "在前端开发过程中，本地运行调试时默认使用 HTTP 协议，而调用某些敏感的功能（比如相机、麦克风、地理位置等）时，浏览器出于安全考虑，只允许在 HTTPS 协议下调用，本文就讲述了如何在本地开发环境中配置 HTTPS。"
tags: ["Tools", "HTTPS", "Vite"]
---
我最近在开发一个能够提供扫码功能的网页，但是扫码功能需要调用相机权限，浏览器出于安全考虑只在 HTTPS 环境下允许。所以想要在本地调试这些功能，需要为本地开发服务器配置 HTTPS。解决这个问题的本质就是**生成本地证书和自签名**，网上能找到很多方法，在这里我会介绍两种，看到这篇博客的朋友可以跟着下面的步骤实践，足够你在简单基础的开发中去使用。

## mkcert
[mkcert](https://github.com/FiloSottile/mkcert) 是一个简化了的本地 CA 工具，用来生成有效的本地 HTTPS 证书。`mkcert` 的使用和证书配置步骤如下：

### 1.安装
如果你也是 macOS 系统且安装了 homebrew，可以按照下面这个方式进行安装：
```shell
brew instal mkcert
brew install nss # 如果电脑中有 Firefox，还需要这一步骤，否则会报错
```
如果你是 Windows，可以[点击此处](https://github.com/FiloSottile/mkcert?tab=readme-ov-file#windows)查看安装步骤。

### 2.安装本地 CA
```shell
mkcert -install
```

### 3.为 localhost 创建证书
```shell
mkcert localhost 127.0.0.1 ::1
```

创建成功后，Terminal 会显示证书的存放位置和过期时间，如下：
```shell
The certificate is at "./localhost+2.pem" and the key at "./localhost+2-key.pem" ✅

It will expire on 25 March 2026 🗓
```

### 4.将证书存放在项目中
在项目目录下新建一个文件夹，方便管理证书。

例如我在我的项目目录下新建了一个 `cert` 的文件夹，并将刚才生成的 `localhost+2.pem` 和 `localhost+2-key.pem` 移动到该目录下。

![存放证书](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202312252213449.png)

### 5.配置 vite 以使用 HTTPS
我的项目技术栈是 `vue3 + vite`，所以这里配置的是 `vite.config.js` 文件，如果你使用其他构建工具(比如`vue-cli + webpack`)，配置方法也都大同小异，只需查阅官方文档做出相应调整即可。
注意：修改配置后记得重新运行服务。
```js
// vite.config.js
import fs from 'fs';
import path from 'path';

export default defineConfig({
  // ...
  server: {
    // ...
    https: {
      key: fs.readFileSync(path.resolve(__dirname, 'cert/localhost+2-key.pem')),
      cert: fs.readFileSync(path.resolve(__dirname, 'cert/localhost+2.pem')),
    },
  },
})
```

## Vue / React 自带选项
不管是 React 还是 Vue，都自带了开启 HTTPS 的选项，这也使得我们在本地使用 HTTPS 协议更加简单。

也以 vite 为例，[官方文档](https://vitejs.dev/config/server-options.html#server-https)中有提到，如果需要合法证书，可以使用它们提供的插件 [@vitejs/plugin-basic-ssl](https://www.npmjs.com/package/@vitejs/plugin-basic-ssl)，这个插件会自动创建和缓存一个自签名的证书。

### 1.安装插件 `@vitejs/plugin-basic-ssl`：
```shell
pnpm add @vitejs/plugin-basic/ssl -D
```

### 2.配置 vite
配置 vite 后运行即可，需要注意浏览器可能会有安全警告，这是因为我们生成的自签名证书不是受信任的证书颁发机构签发的，但不妨碍我们开发使用，只要不投入生产环境使用就没问题。
```js
// vite.config.js
export default defineConfig({
  // ...
  plugins: [
    // ...
    basicSsl(),
  ],
  server: {
    // ...
    https: true,
  },
})
```