---
title: 'Docker Proxy Configuration (Resolving Login Issues)'
date: 2024-06-05
description: "Solving Docker Desktop login failure issues"
summary: "Solving Docker Desktop login failure issues"
tags: ["Docker"]
---

> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

Docker Desktop login failures can occur for many reasons. If you can log in to [Docker Hub](https://hub.docker.com/) via web browser without issues but Docker Desktop login times out, it's likely due to proxy settings. You can follow the method below to configure it.

## Login Issues

First, you need to have a proxy (Google it yourself) and get its port number. For example, I use Clash, and in its settings, I can see the port number is 7897.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406051122307.png)

Open Docker Desktop settings, go to Resource -> Proxies.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406051130481.png)

Enable `Manual Proxy configuration`, add Web Server(HTTP) and Web Server(HTTPS), using your proxy port number.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406051132566.png)

Restart Docker Desktop and you should be able to log in normally.

## Image Acceleration

When pulling images from Docker Hub in China, you might encounter difficulties. You can refer to [China's Docker Hub Mirror Accelerators](https://gist.github.com/y0ngb1n/7e8f16af3242c7815e7ca2f0833d3ea6) to configure image acceleration.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202406051148187.png)