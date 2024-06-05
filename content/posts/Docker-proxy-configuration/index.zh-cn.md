---
title: 'Docker 代理配置（解决登录失败问题）'
date: 2024-06-05
description: "解决 Docker Desktop 登录失败的问题"
summary: "解决 Docker Desktop 登录失败的问题"
tags: ["Docker"]
---

Docker Desktop 登录失败有很多原因，如果使用网页打开 [Docker Hub](https://hub.docker.com/) 登录没问题，但是 Docker Desktop 登录超时，大概率是因为代理问题，可以参考下面的方法进行配置。

首先你要有一个代理（自行 Google），获取到代理的端口号。例如我使用的是 Clash，在设置中可以看到端口号为 7897。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406051122307.png)

打开 Docker Desktop 设置，Resource -> Proxies。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406051130481.png)

打开 `Manual Proxy configuration` 状态，添加 Web Server(HTTP) 和 Web Server(HTTPS)，端口号为代理端口号。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406051132566.png)

重启 Docker Desktop 就可以正常登录了。