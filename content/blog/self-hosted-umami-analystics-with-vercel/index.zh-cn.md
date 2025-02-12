---
title: "使用 Vercel 部署自托管 Umami 实时监控网站"
date: 2025-02-11T09:14:16+08:00
description: " "
---

> Umami is a simple, fast, privacy-focused alternative to Google Analytics.

Umami 是一个简单、快速、注重隐私的网站分析工具，旨在替代 Google Analytics。它提供实时的网站访问统计、用户行为分析等功能，帮助网站管理员更好地了解网站的访问情况和用户行为。

本文将详细介绍如何在 Vercel 上免费部署 umami，并且使用 Vercel 创建一个免费的数据库。

### 通过 Vercel 免费部署 Umami

1. 在 Github Fork umami 项目
   fork https://github.com/umami-software/umami 到自己的 github 账号
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111620557.png)
2. 在 Vercel 导入 umami 项目
   打开 https://vercel.com，如果没有账号需要先注册，来到 Overview 面板，点击 `Add New`，选择 Project
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111645958.png)
   选择刚刚 fork 的 umami 项目，点击 `Import`
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111646955.png)

   进入 New Project 页面，Environment Variables 是必需的，需要添加 DATABASE_URL，这些值在项目初始安装中会被用来作为 umami 配置。先空着不填，稍后我们会同样使用 Vercel 创建一个免费的数据库。
   
   此处直接点击 `Deploy` 部署即可，由于没有提供环境变量，Vercel 会提示部署失败。
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111652854.png)
3. 在 Vercel 创建 Neon 数据库并关联
   切换到 Storage 面板，点击 `Create Database`
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111703029.png)
  umami 官方文档要求使用 PostgreSQL，所以此处我们选择 [Neon](https://vercel.com/marketplace/neon) 。
  ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111706699.png)
  我的配置
  ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111711000.png)
  创建成功后，进入数据库页面，点击 `Connect Project`
  ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111712220.png)
  连接刚才创建好的 umami 项目
  ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111714214.png)
1. 重新部署 umami 项目
   连接好数据库后，就可以重新部署了，点击 umami 项目，进入 Deployments 面板，找到刚才部署失败的 deployment，点击 `Redeploy` 重新部署，成功部署大约需要几分钟时间。
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502111721494.png)

### 使用 umami 监控网站
部署成功后，登录 umami 后台。初始账号密码：
- UserName: admin
- Password: umami

1. 首次登录先修改密码
2. 点击 Settings 面板，选择 Websites，点击 `Add Website` 按钮添加需要监控的域名。
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502120958796.png)
3. 为需要监控的网站添加 tracking code
   点击 Edit，进入刚刚添加的网站，将这段 `script` 添加到网站的 `<head></head>` 中。
   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502121000976.png)
   以 hugo 为例，我使用 [Hugo Module](https://gohugo.io/hugo-modules/) 管理主题 module [hextra](https://github.com/imfing/hextra)，无法直接修改主题文件。

   我的做法是直接复制一份 hextra 的 `layouts/partials/head.html` 文件到自己的 hugo 项目中，添加上 umami 提供的 tracking code。

   ![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202502120952653.png)