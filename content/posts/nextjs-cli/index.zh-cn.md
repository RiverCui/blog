---
title: 'Next.js CLI'
date: 2024-01-15
draft: false
tags: ["Next.js", "React"]
series: ["Next.js"]
series_order: 1
---
{{<admonition question "什么是 Next.js">}}
[Next.js](https://nextjs.org) 是基于  `Node.js` 和 `React` 的 Web 开发框架，集成了基于 `Rust` 的 `JavaScript` 工具，可以快速创建**全栈应用**。
{{</admonition>}}
{{<admonition question "什么是 Next.js CLI">}}
**CLI** (Command Line Interface) 即命令行界面，是一种用**文本命令**与计算机系统**交互**的方式，用户可以通过在终端输入特定命令来执行操作。

[Next.js CLI](https://nextjs.org/docs/pages/api-reference/next-cli) 是用来帮助用户启动、构建和导出应用程序的，是我们开发中常用到的命令行。
{{</admonition>}}

不过在讲 Next.js CLI 之前，我们先来创建一个演示项目吧！

## 创建一个 Next app
### 1. 自动创建

打开终端，执行下方命令：

```Bash
npx create-next-app@latest
```

执行上面这行命令后，需要去选择下方的这些配置，作为一个演示项目，全部选择默认选项即可。

```Bash
What is your project named?  my-app
Would you like to use TypeScript?  No / Yes
Would you like to use ESLint?  No / Yes
Would you like to use Tailwind CSS?  No / Yes
Would you like to use `src/` directory?  No / Yes
Would you like to use App Router? (recommended)  No / Yes
Would you like to customize the default import alias (@/*)?  No / Yes
```

回答完上面这些 prompts，就会根据你刚刚的配置自动的生成一个新的项目。

---

除了上面这种**交互式**的选择方式，我们也可以通过传递命令行参数，从而以**非交互式**的途径去创建一个新项目。

```Bash
// 查看构建项目时可用的命令行参数
npx create-next-app --help

// 打印如下
Usage: create-next-app <project-directory> [options]
 
Options:
  -V, --version                        output the version number
  --ts, --typescript
 
    Initialize as a TypeScript project. (default)
 
  --js, --javascript
 
    Initialize as a JavaScript project.
 
  --tailwind
 
    Initialize with Tailwind CSS config. (default)
 
  --eslint
 
    Initialize with ESLint config.
 
  --app
 
    Initialize as an App Router project.
 
  --src-dir
 
    Initialize inside a `src/` directory.
 
  --import-alias <alias-to-configure>
 
    Specify import alias to use (default "@/*").
 
  --use-npm
 
    Explicitly tell the CLI to bootstrap the app using npm
 
  --use-pnpm
 
    Explicitly tell the CLI to bootstrap the app using pnpm
 
  --use-yarn
 
    Explicitly tell the CLI to bootstrap the app using Yarn
 
  --use-bun
 
    Explicitly tell the CLI to bootstrap the app using Bun
 
  -e, --example [name]|[github-url]
 
    An example to bootstrap the app with. You can use an example name
    from the official Next.js repo or a public GitHub URL. The URL can use
    any branch and/or subdirectory
 
  --example-path <path-to-example>
 
    In a rare case, your GitHub URL might contain a branch name with
    a slash (e.g. bug/fix-1) and the path to the example (e.g. foo/bar).
    In this case, you must specify the path to the example separately:
    --example-path foo/bar
 
  --reset-preferences
 
    Explicitly tell the CLI to reset any stored preferences
 
  -h, --help                           output usage information
```

以 `-e, --example` 参数为例说一下如何使用：

Next.js 提供了丰富的[示例代码](https://github.com/vercel/next.js/tree/canary/examples)，例如 `with-redux`、`api-routers-cors`、`with-electron`、`with-jest`、`with-markdown`、`with-material-ui`、`with-mobx`，这些代码演示了 Next.js 的各种使用场景，能够让我们快速学习上手。

比如你想查看 `with-redux` 的使用方法，无需去 clone 代码，直接在创建项目的时候使用 `--example` 即可：

```Bash
npx create-next-app --example with-redux your-app-name
```

{{<image src="https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401092224194.png" caption="官方 with-redux demo">}}

### 2. 手动创建
你也可以选择手动创建项目：
```Bash
npm install next@latest react@latest react-dom@latest
```
npm 会自动创建 `package.json` 文件并且安装依赖项，但你还需要手动添加 `scripts`：

```json
// package.json
{
  ...
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
  ...
}
```
{{<admonition tip>}}
如果是全新的项目，推荐**自动创建方式**。如果是项目中引入 Next.js，可以参考手动创建方式。
{{</admonition>}}

## Next.js CLI

我们在文章的开头讲过了 Next.js CLI 是什么，至于 Next.js CLI 到底有哪些命令，可以通过 `npx next --help` 快速查看命令列表：

```Bash
Usage
      $ next <command>

    Available commands
      build, start, export, dev, lint, telemetry, info, experimental-compile, experimental-generate

    Options
      --version, -v   Version number
      --help, -h      Displays this message

    For more information run a command with the --help flag
      $ next build --help
```

### Build 构建
#### `next build`
执行 `next build` 会创建项目**生产优化版本**。

```Bash
npx next build
```

构建时会输出每条路由的信息，如下图所示：

![next build output](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401101552921.png)

可以看到，每条路由对应两条数据，分别是 `Size` 和 `First Load JS`。
- `Size` 表示目标路由单独依赖项的 JS 大小。
- `First Load JS` 表示加载目标路由一共所需的 JS 大小。

也许你已经注意到，上图中还有一项数据 `First Load JS shared by all`，是指每个路由都需要依赖的 JS 大小。也就是说 `Size`+`First Load JS shared by all`=`First Load JS`。

{{<admonition>}}
这些值都是 gzip 压缩后的大小，`First Load JS` 会用绿色、黄色和红色来表示，**绿色代表着高性能**，黄色和红色则需要优化。
{{</admonition>}}


#### `next build --profile`

这个命令用于开启 React 的**生产性能分析**（需要 Next.js v9.5 以上）。在使用这个功能前，先为你的浏览器安装 [React 开发者工具](https://chromewebstore.google.com/detail/react-developer-tools)。

> `Profiler` 是 React 中测量渲染性能的 API，你可以[点击此处](https://react.dev/reference/react/Profiler)了解更多。 

修改 `page.js` 添加 Profiler Component:

```js
import React from 'react';

export default function Page() {
  return (
    <React.Profiler id="Hello">
      <p>Hello World!</p>
    </React.Profiler>
  )
}
```

在终端执行 `npm run dev`，可以在浏览器控制台看到：
![React Developer Tools Profiler](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401131005879.png)

但是执行 `npm run build` 和 `npm run start`，浏览器控制台查看会提示 —— 不支持 `profiling`。
![React Developer Tools Profiler](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401131033233.png)

这是因为 Profiler 需要 React v16.5+ 的开发模式或者 `profiling build`。这需要我们在构建的时候加上 `--profile`，所以执行 `npx next build --profile` 和 `npm run start` 再试一次：

![React Developer Tools Profiler](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401131043860.png)

{{< style "strong{color:#61bc84;}" >}}
可以看到 Profiler 已经正常运行了，**这个功能可以帮助我们排查线上的性能问题**。
{{< /style >}}


#### `next build --debug`
使用这个参数获取更详细的构建输出，比如 `rewrites`、`redirect`、`headers`。

修改 `next.config.js`，添加 `redirects` 重定向和 `rewrites` 重写：
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  async redirects() {
    return [
      {
        source: '/index',
        destination: '/',
        permanent: true,
      },
    ]
  },
  async rewrites() {
    return [
      {
        source: '/about',
        destination: '/',
      },
    ]
  }
}
module.exports = nextConfig
```
执行 `npx next build --debug`，输出结果如下：
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202401161159531.png)

可以看到，输出信息多了 `redirects` 和 `rewrites` 的内容。

### Development 开发模式
#### `next dev`
`next dev` 在开发模式下启动应用程序，包括热加载、错误报告等功能。

应用默认在 http://localhost:3000 启动，也可以使用 `-p` 来修改默认端口，例如：
```Bash
npx next dev -p 4000
```

你也可以修改主机名(hostname)取代默认的 `0.0.0.0`，以便其他设备访问，使用 `-H` 来修改 hostname，例如：
```Bash
npx next dev -H 192.168.1.2
```

#### 本地开发配置 HTTPS
有某些用例比如 webhooks 或者身份验证，浏览器出于安全考虑要求使用 HTTPS，`next.js` 可以使用 `next dev` 生成自签名证书，如下所示：
```Bash
next dev --experimental-https
```

也可以提供自定义证书和密钥：
```Bash
next dev --experimental-https --experimental-https-key ./certificates/localhost-key.pem --experimental-https-cert ./certificates/localhost.pem
```

{{<admonition>}}
`next dev --experimental-https` 只是用 `mkcert` 创建一个用于开发模式的本地受信任证书，在生产环境中，需要使用受信任机构办法的证书。当然如果你在 `Vercel` 上部署，`Vercel` 会自动为你配置 Next.js 应用。
{{</admonition>}}

在写这篇文章之前，我也遇到过要求 HTTPS 的场景（浏览器需要调用相机权限），并且专门写了文章介绍如何 [为本地开发服务器配置 HTTPS](../https-for-local-development/)。

### Production 生产模式
`next start` 以生产模式启动应用程序，应用需要先使用 `next build` 编译。

应用默认在 http://localhost:3000 启动，可以通过 `-p` 修改端口号，例如：

```Bash
npx next start -p 4000
```

### Lint 代码质量检查
`next lint` 为`pages/`、`app/`、`components/`、`lib/`和`src/` 目录下的所有的文件运行 ESlint。

如果你还有其他文件夹要进行代码检查，可以通过 `--dir` 指定文件夹：
```Bash
npx next lint --dir utils
```

### Info 系统信息
`next info` 打印关于当前系统的相关详情，用于报告 Next.js 的 bug。

在项目根目录运行：
```Bash
next info
```
会得到类似的信息：
```
Operating System:
  Platform: linux
  Arch: x64
  Version: #22-Ubuntu SMP Fri Nov 5 13:21:36 UTC 2021
Binaries:
  Node: 16.13.0
  npm: 8.1.0
  Yarn: 1.22.17
  pnpm: 6.24.2
Relevant packages:
  next: 12.0.8
  react: 17.0.2
  react-dom: 17.0.2
```

然后将此信息粘贴到GitHub问题中，以供官方人员进行问题排查。
