---
title: 'Next.js 路由 App Router'
date: 2024-01-15
draft: true
description: '目前 Next.js 的路由方案有两种—— pages router 和 app router，本篇文章介绍了两种路由的使用方式'
tags: ["Next.js", "React"]
series: ["Next.js"]
series_order: 2
---

Next.js 目前有两种路由方案，一个是旧有的 [Pages Router](https://nextjs.org/docs/pages)，另一个是 13 版本新推出的 [App Router](https://nextjs.org/docs/app)。在这篇文章中，我们将深入探讨 **App Router 的核心概念、如何在 Next.js 项目中实现高效路由管理，以及 App Router 如何改变我们对 Web 应用路由的理解**。无论你是一位经验丰富的 Next.js 开发者，还是刚刚开始接触这个框架的新手，本文都将为你揭示 App Router 的强大功能和潜力。

## App Router 和 Pages Router

App Router 在 `app` 目录下工作，Pages Router 在 `page` 目录下工作。

为了能够逐步采纳 App Router，两套方案是兼容的，也就是说 `app` 目录与 `pages` 目录可以同时存在，所以过去使用 Pages Router 范式的项目，在新增路由的时候也能选择用 App Router 或 Pages Router。

tips: **App Router 优先级高于 Pages Router**，且跨目录的路由不能解析到相同的 URL 路径上，构建时会报错。

`app` 目录下的页面默认使用 `React Server Component`，这是一项性能优化，不过你也可以通过声明 `use client` 使用 `Client Component

相较之下，App Router 功能更强、性能更好、代码组织更灵活，对于新应用官方更推荐使用 App Router。

## 使用 App Router

Next. js 使用基于 file-system (文件系统)的路由器，所以我们要关注两个元素：
- folders 文件夹
- file 文件

### 定义路由

文件夹用来定义路由，每个文件夹都代表一个对应 URL 片段的路由片段。

如下图所示，要创建一个嵌套路由，就可以通过嵌套文件夹来实现，`app/dashboard/settings` 目录对应的路由地址就是 `dashboard/settings`：

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221134740.png)

### 定义页面 Pages

`page.js` 文件用来定义页面。
  
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221135459.png)

比如这个例子中，`dashboard/analytics` 的 URL 路径就不能被访问，因为它没有相应的 `page.js` 文件。这个文件夹可以用来存储组件、样式表、图像或其他托管文件。

现在我们来创建第一个 `page.js` 页面吧：
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221146387.png)

```js
// app/page.js
export default function Page() {
  return <h1>Hello, Home Page!</h1>
}
```

```js
// app/dashboard/page.js
export default function Page() {
  return <h1>Hello, Dashboard Page!</h1>
}
```

`npm run dev` 运行项目，访问 `http://localhost:3000/`，效果如下：
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221149969.png)

访问 `http://localhost:3000/dashboard/`：
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221149725.png)

tips: 不止 `.js` 文件，Next.js 默认支持 React、TypeScript，所以 `.js`、`.jsx`、`.tsx` 文件都是可以的。

### 定义布局 Layouts

布局是在多个页面之间**共享**的UI。在导航中，布局保留状态，保持交互式，并且**不会重新渲染**。布局也可以嵌套。

现在新建一个 `layout.js` 的文件，该文件会默认导出一个 React 组件，这个组件接收一个 `children` prop，`children` 用来表示子布局（如果有的话）或者子页面。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221157258.png)

```js
// app/dashboard/layout.js
export default function DashboardLayout({
  children,
}) {
  return (
    <section>
      <nav>NAV</nav>
      {children}
    </section>
  )
}
```

效果如图所示：
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221200578.png)

其中，`NAV` 来自于 `app/dashboard/layout.js`，`Hello, Dashboard Page!` 来自于 `app/dashboard/page.js`。

**在同一文件夹下，如果有 `layout.js` 和 `page.js`，`page.js` 会自动作为 `children` 参数传入 `layout.js`。**也就是说，**layout 会包裹同层级的 page**。
