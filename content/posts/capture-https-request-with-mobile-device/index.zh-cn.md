+++
title = '抓取移动设备的 HTTPS 请求'
date = 2024-03-10T17:36:56+08:00
tags = ["Tools", "HTTPS"]
+++

最近在为公司重构项目，因为原本是原生安卓和ios应用，所以为了快速了解业务逻辑和编写新代码，我决定一边用手机操作业务进行抓包，一边对照已有文档、代码进行重构。这篇文章介绍了我是**如何用一台电脑和手机来实现抓包移动设备网络请求的**。

首先介绍一下使用的工具 —— **Charles**，这是一款网络抓包工具，它允许开发者查看、监控发送和接收的HTTP/HTTPS通信数据，主要面向开发者对 Web应用、移动 App 等进行网络调试和分析。你可以点击进入 [Charles 官网](https://www.charlesproxy.com/)下载，支持 Windows、macOS 和 Linux。

### 基本使用

安装好 Charles 后，可以按照我下面给出的步骤进行使用：

#### 1. 让你的手机和电脑处在同一局域网

#### 2. 电脑运行 Charles（默认端口号 8888）

#### 3. 打开手机的 WLAN 设置

修改以下三项，可参考下方小米手机截图：

- 代理：改为**手动**

- 主机名：填写你的**电脑 IP**

- 端口号： **8888**（8888 为 Charles 默认端口号，如有需要可以在 Charles 中的 Proxy Settings 修改）

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141801827.png)

#### 4. 在手机中打开需要抓取的 App

现在你可以看到 Charles 已经开始正常运行，开始抓取 HTTP 请求了。

虽然 HTTP 请求抓取正常，但你也许会和我一样，有需要抓取 **HTTPS 请求**的使用场景，但此时 Charles 抓包会出现乱码，这是**由于 HTTPS 更高的安全性**，我们要提供证书才可以正常获取数据。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141813564.png)

### 配置以抓包 HTTPS Request

在移动设备上安装证书：

#### 1. 配置 Charles 的 SSL

Proxy -> SSL Proxying Settings

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141820500.png)

添加 `*:443`。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141818660.png)

#### 2. 从 Charles 下载证书

打开 Charles，Help -> SSL Proxying，因为我们使用的是手机，所以这里就选择 Mobile Device。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141824077.png)

Charles 会弹出提示，我们根据提示，用手机访问 `chls.pro/ssl` 下载证书。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141826786.png)

#### 3. 手机安装证书

还是以小米手机为例，系统是 MUI 13.0.7，安装证书步骤如下：

安全 -> 更多安全设置 -> 加密与凭据 -> 安装证书 -> CA 证书，然后选择刚才下载好的 Charles 证书安装

现在再打开 Charles，可以看到抓到的 HTTPS 请求信息了。

