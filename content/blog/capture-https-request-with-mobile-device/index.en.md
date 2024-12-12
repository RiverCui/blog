---
title: 'Capturing Mobile Device HTTPS Requests with Charles'
date: 2023-03-10T17:36:56+08:00
description: "Introduction to using Charles proxy tool to capture HTTPS requests from mobile devices"
summary: "A detailed guide on how to use Charles proxy tool with a computer and mobile phone to capture HTTP and HTTPS requests from mobile apps."
tags: ["Tools", "HTTPS"]
---

> Note: This article was translated from Chinese to English by Claude AI (Anthropic).

Recently, while working on a project reconstruction for my company, which originally had native Android and iOS applications, I decided to understand the business logic and write new code by capturing network packets while operating the business on a mobile phone, referencing existing documentation and code for reconstruction. This article explains **how I used a computer and a mobile phone to capture network requests from mobile devices**.

First, let me introduce the tool — **Charles**, a network packet capture tool that allows developers to view and monitor HTTP/HTTPS communication data, primarily designed for developers to debug and analyze web applications and mobile apps. You can visit the [Charles website](https://www.charlesproxy.com/) to download it, which supports Windows, macOS, and Linux.

### Basic Usage

After installing Charles, you can follow these steps:

#### 1. Ensure your phone and computer are on the same local network

#### 2. Run Charles on your computer (default port 8888)

#### 3. Open WLAN settings on your phone

Modify the following three items, referring to the Xiaomi phone screenshot below:

- Proxy: Change to **Manual**

- Hostname: Enter your **computer's IP**

- Port: **8888** (8888 is Charles' default port, which can be modified in Charles' Proxy Settings if needed)

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141801827.png)

#### 4. Open the app you want to capture on your phone

Now you can see Charles running normally and starting to capture HTTP requests.

Although HTTP request capture works fine, you might, like me, need to capture **HTTPS requests**. At this point, Charles will show scrambled data due to **HTTPS's higher security**. We need to provide a certificate to properly access the data.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141813564.png)

### Configuring to Capture HTTPS Requests

Installing certificates on mobile devices:

#### 1. Configure Charles SSL

Proxy -> SSL Proxying Settings

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141820500.png)

Add `*:443`.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141818660.png)

#### 2. Download Certificate from Charles

Open Charles, Help -> SSL Proxying, and since we're using a phone, select Mobile Device here.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141824077.png)

Charles will show a prompt. Following the instructions, use your phone to visit `chls.pro/ssl` to download the certificate.

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202405141826786.png)

#### 3. Install Certificate on Phone

Taking a Xiaomi phone as an example, running MIUI 13.0.7, here are the steps to install the certificate:

Security -> More Security Settings -> Encryption & Credentials -> Install Certificate -> CA Certificate, then select and install the Charles certificate you just downloaded

Now when you open Charles, you can see the captured HTTPS request information.