---
title: 'ChatGPT 如何处理流式数据'
date: 2025-02-08
description: "本文将展示 ChatGPT API 如何使用 SSE（服务器端事件）实时流式传输生成中的响应，以及如何使用 Node.js 和 express 实现简单SSE。"
---

## SSE 是什么

SSE 即 Server-sent events，可以翻译为服务端事件。

简单来说，SSE 是一个请求和非常长的响应。

它允许服务端在数据可用时立即推送到客户端，而无需客户端不断询问或轮询服务器以获取新数据。这允许在不需要持续请求的情况下将实时更新传递给客户端。

## 为什么 ChatGPT 会选择 SSE

首先我们要清楚，客户端与服务端通信既可以通过客户端主动请求，也可以通过服务器主动推送。

### 客户端主动请求

1. 普通请求(Simple Request)
  - 一次请求一次响应
  - 最基本的 HTTP 请求/响应模式
2. 轮询(Polling)
  - 客户端定期发送请求查询更新
  - 固定时间间隔，不管有无数据变化都请求
3. 长轮询(Long Polling)
  - 客户端发起请求，服务器挂起连接直到有数据更新
  - 服务器响应后，客户端立即发起下一次请求

### 服务器推送技术

1. WebSocket
  - 全双工通信
  - 需要客户端和服务器都支持 WebSocket 协议
  - 适合实时性要求高的场景（如聊天、游戏）
  
2. SSE (Server-Sent Events)
  - 服务器到客户端的单向通信
  - 基于 HTTP 协议
  - 适合服务器数据推送场景（如通知、实时数据流）

3. HTTP/2 Server Push

  - 服务器主动推送相关资源
  - 通常用于推送静态资源
  - 需要 HTTP/2 支持

若要实现 ChatGPT 中的实时通信，可以通过 SSE 或 WebSocket，为什么 ChatGPT 会选择 SSE 而不是 WebSocket 呢？

我们来仔细对比下 SSE 和 WebSocket 的不同点：
| 特性       | SSE          | WebSocket   |
|----------|--------------|-------------|
| 通信方向   | 单向         | 双向        |
| 协议       | HTTP/HTTPS   | WS/WSS      |
| 自动重连   | 支持         | 需手动实现  |
| 最大连接数 | 受浏览器限制 | 较高        |
| 数据格式   | 文本         | 文本/二进制 |
| 跨域支持   | 需要 CORS    | 原生支持    |
| 实现复杂度 | 简单         | 相对复杂    |
| 服务器压力 | 较小         | 较大        |

SSE 使用场景：
- 实时通知推送
- 股票行情更新
- 社交媒体 feed 流
- 日志实时显示

WebSocket 使用场景：
- 在线聊天室
- 多人游戏
- 协同编辑
- 实时数据分析

考虑到以下几点，ChatGPT 选择 SSE 足够：

1. 单向通信。ChatGPT 只需要从服务端到客户端实时推送，并不需要频繁的双向通信，SSE 更符合需求。
2. 实现和维护复杂度。SSE 简单易用，只需要服务端推送消息，客户端通过标准的 `EventSource` API 可以轻松接收消息，并且是基于 HTTP/HTTPS 实现，不需要像 WebSocket 那样进行复杂的连接握手和状态管理。
3. 兼容性和可靠性。SSE 通过 HTTP/1.1 实现，能够更好的穿透代理服务器、防火墙等设施，保证了消息推送可靠性。WebSocket 需要协议从 HTTP 升级到 WebSocket 协议(WS 或 WSS)，某些网络可能会阻断这种升级过程，从而影响连接的可靠性。
4. 自动重连和消息重发。SSE 支持自动重连功能，WebSocket 重连需要手动实现较复杂。
5. 资源效率和性能。SSE 使用较少的资源，服务器压力小，WebSocket 性能较优但资源消耗大，对服务器来说压力更大。
6. 使用场景的适配性。SSE 适合更低频的消息推送，WebSocket 适合高频双向通信。

在 ChatGPT 的情况下，当你发送消息时，服务器可能会立即开始使用机器学习算法生成响应。一旦服务器生成新的文本，他就可以通过 SSE 发送给客户端，这样就允许客户端在响应到达时立即渲染。

## SSE Demo
数据流效果：
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/Kapture%202025-02-08%20at%2001.10.03.gif)

可以看到response一直在变化：
![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/Kapture%202025-02-08%20at%2001.12.07.gif)

源码：[RiverCui/sse-demo](https://github.com/RiverCui/sse-demo)
