---
title: 'Next.js Routing: App Router'
date: 2024-01-15
draft: true
tags: ["Next.js", "React"]
series: ["Next.js"]
series_order: 2
---
> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

Next.js currently offers two routing solutions: the legacy [Pages Router](https://nextjs.org/docs/pages) and the new [App Router](https://nextjs.org/docs/app) introduced in version 13. In this article, we will **dive deep into the core concepts of App Router, explore how to implement efficient routing management in Next.js projects, and understand how App Router transforms our approach to Web application routing**. Whether you're an experienced Next.js developer or just starting with this framework, this article will reveal the powerful features and potential of App Router.

## App Router vs Pages Router

App Router operates in the `app` directory, while Pages Router works in the `pages` directory.

To enable gradual adoption of App Router, both solutions are compatible - meaning the `app` and `pages` directories can coexist. This allows projects previously using the Pages Router paradigm to choose between App Router or Pages Router when adding new routes.

tip: **App Router takes precedence over Pages Router**, and routes across directories cannot resolve to the same URL path - this will result in a build error.

Pages in the `app` directory use `React Server Component` by default for performance optimization. However, you can also use `Client Component` by declaring `use client`.

Comparatively, App Router offers more functionality, better performance, and more flexible code organization, making it the recommended choice for new applications.

## Using App Router

Next.js uses a file-system based router, so we need to focus on two elements:
- folders 
- files

### Defining Routes

Folders are used to define routes, with each folder representing a route segment corresponding to a URL segment.

As shown in the image below, nested routes can be created through nested folders. The `app/dashboard/settings` directory corresponds to the route path `dashboard/settings`:

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221134740.png)

### Defining Pages

The `page.js` file is used to define pages.
  
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221135459.png)

For example, the URL path `dashboard/analytics` cannot be accessed because it doesn't have a corresponding `page.js` file. This folder can be used to store components, stylesheets, images, or other collocated files.

Let's create our first `page.js` pages:
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

Run `npm run dev` and visit `http://localhost:3000/`, you'll see:
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221149969.png)

Visit `http://localhost:3000/dashboard/`:
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221149725.png)

tip: Not just `.js` files - Next.js supports React and TypeScript by default, so `.js`, `.jsx`, and `.tsx` files are all acceptable.

### Defining Layouts

Layouts are UI components that are **shared** between multiple pages. During navigation, layouts preserve state, remain interactive, and **do not re-render**. Layouts can also be nested.

Let's create a `layout.js` file. This file exports a default React component that accepts a `children` prop. The `children` prop represents child layouts (if any) or child pages.

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

The result looks like this:
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401221200578.png)

Here, `NAV` comes from `app/dashboard/layout.js`, and `Hello, Dashboard Page!` comes from `app/dashboard/page.js`.

**In the same folder, if there are both `layout.js` and `page.js` files, `page.js` will automatically be passed as the `children` parameter to `layout.js`.** In other words, **the layout wraps the page at the same level**.