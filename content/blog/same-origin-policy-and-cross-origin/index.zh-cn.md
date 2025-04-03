---
title: "浏览器同源策略和跨域"
date: 2025-04-03T08:59:56+08:00
description: "详细介绍浏览器的同源策略及其限制，并探讨多种跨域解决方案，包括CORS、代理服务器、JSONP、postMessage等常用技术的实现方法和最佳实践。"
draft: true
---

## 什么是浏览器同源策略

> [!note] 同源策略(Same Origin Policy)
> **同源策略：protocol（协议）、domain（域名）、port（端口）三者必须一致。**
> 
> 同源策略限制了从同一个源加载的文档或脚本如何与另一个源的资源进行交互。这是浏览器的一个用于隔离潜在恶意文件的重要的安全机制。

举个例子，下表给出了与 URL [http://store.company.com/dir/page.html](http://store.company.com/dir/page.html) 的源进行对比的示例:

| URL                                             | 是否跨域 | 原因                      |
| ----------------------------------------------- | ---- | ----------------------- |
| http://store.company.com/dir/page.html          | 同源   | 完全相同                    |
| http://store.company.com/dir/inner/another.html | 同源   | 只有路径不同                  |
| https://store.company.com/secure.html           | 跨域   | 协议不同                    |
| http://store.company.com:81/dir/etc.html        | 跨域   | 端口不同 ( http:// 默认端口是80) |
| http://news.company.com/dir/other.html          | 跨域   | 域名不同                    |

**同源策略主要限制了三个方面：**

- 当前域下的 **js 脚本**不能够访问其他域下的 cookie、localStorage 和 indexDB。
- 当前域下的 **js 脚本**不能够访问操作其他域下的 DOM。
- 当前域下 **ajax** 无法发送跨域请求。

同源政策的目的主要是为了保证用户的信息安全，它只是对 **js 脚本**的一种限制，并不是对浏览器的限制，对于一般的 img、或者script 脚本请求都不会有跨域的限制，这是因为这些操作都不会通过响应结果来进行可能出现安全问题的操作。

跨域问题其实就是浏览器的同源策略造成的。

不管是在开发环境还是生产环境中，跨域都是一个经常遇到的问题。为了解决跨域问题，有以下几种常用的解决方案：

## 跨域解决方案

### 1. 开发环境使用代理服务器

> [!tip] 原理
> 跨域限制是**浏览器**的一种安全措施，而服务器与服务器之间的通信不受限制。
> 在开发环境中，配置代理服务器，浏览器发送请求到代理服务器，服务器再将请求转发到实际的 API 服务器。API 服务器返回响应到代理服务器，代理服务器再返回给浏览器。
> 这个过程实际上是让**代理服务器**代替**浏览器**向**目标服务器**发送请求，不受同源策略影响。

> [!attention] 注意
> 这种配置**只**存在于开发环境，在**生产环境**中，我们需要在真正的 web 代理服务器（如 **`nginx`**）上配置反向代理。

#### Webpack

在 `webpack.config.js` 中配置：

```JavaScript
module.exports = {
	// ...其他配置
	devServer: {
		proxy: {
			'/api': {
				target: 'http://api.example.com',
				changeOrigin: true,
				pathRewrite: { '^/api': '' },
			}
		}
	}
}
```

#### Vite

在 `vite.config.js` 中配置：

```JavaScript
export default {
	server: {
		proxy: {
			'/api': {
				target: 'http://api.example.com',
				changeOrigin: true,
				rewirte: path => path.replace(/^\/api/, ''),
			}
		}
	}
}
```

#### Create React App

在 `package.json` 中添加：

```json
{
	"proxy": "http://api.exmaple.com"
}
```

或在 `src/setupProxy.js` 中设置：

```JavaScript
const { createProxyMiddleware } = require('http-proxy-middleware')

module.exports = function(app) {
	app.use(
		'/api',
		createProxyMiddleware({
			target: 'http://api.example.com',
			changeOrigin: true,
			pathRewrite: {
				'^/api': '',
			}
		})
	)
}
```

#### Vue CLI

在 `vue.config.js` 中配置：

```JavaScript
module.exports = {
	devServer: {
		proxy: {
			'/api': {
				target: 'http://api.example.com',
				changeOrigin: true,
				pathRewrite: {
					'^/api': '',
				}
			}
		}
	}
}
```

### 2. CORS

CORS(Cross-Origin Resource Sharing，跨域资源共享)是一种浏览器安全机制，它通过 HTTP 头来告诉浏览器允许某个源（域名）访问来自不同源的资源。

#### 工作原理

> [!tip] 简单请求、非简单请求
> 关于简单请求、非简单请求和预检请求的概念，请查看 [[#5. 简单请求与非简单请求的区别，什么时候会进行预检请求？]]

1. 简单请求：某些请求（如 `GET`、`POST` 等）在满足特定条件时直接发送请求，携带 `Origin` 头。
2. 预检请求：复杂请求先发送 `OPTIONS` 请求（预检），询问服务器是否允许实际请求。
3. 服务器响应：返回特定 CORS 头部，告诉浏览器是否允许跨域请求。

#### 服务器端配置

```JS
// Node.js Express 示例
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'https://allowed-origin.com');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.header('Access-Control-Allow-Credentials', 'true');
  
  // 处理预检请求
  if (req.method === 'OPTIONS') {
    return res.sendStatus(200);
  }
  next();
});
```

> [!note]
> CORS 需要浏览器和服务器同时支持，整个 CORS 过程都是浏览器自动完成，无需用户参与。

##### **实现 CORS 的关键**

```HTML
Access-Control-Allow-Origin: https://example.com
```

##### **处理预检请求必需的**

```HTML
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, X-Request-With
Access-Control-Max-Age: 86400 // 预检请求结果缓存时间
```

其中 `Access-Control-Max-Age` 可以设置预检请求结果的缓存时间，单位是秒，在这个时间范围内，再次发送请求就不需要再进行预检请求了，可以有效减少预检请求的次数。

##### **CORS中Cookie相关问题**

`Access-Control-Allow-Credentials: true` 是与跨域请求中的**凭据传递**相关的头部信息。

> [!tip] 凭据
> 在 Web 中，凭据主要指：
> - Cookie
> - HTTP 认证头（如 Basic 或 bearer 认证）
> - TLS 客户端证书

出于对安全的考虑，浏览器**默认不会**在跨域请求中发送凭据，服务器也不会接收跨域请求中的凭据。这是因为凭据中往往包含用户敏感信息（比如用户会话 ID）。

但在一些场景中，是需要在跨域请求中携带凭据的，例如：

- 维持用户会话状态
- 发送认证 Token（在 Cookie 中）
- 访问需要认证的 API 端点

这个时候就需要做一些特殊处理，以便**在跨域请求中传递凭据**。设置 `Access-Control-Allow-Credentials: true`，有两个作用：

1. 告诉浏览器，**服务器**允许**接收**和**处理**跨域请求中的**凭据**
2. 允许**浏览器**将该**响应数据**提供给包含凭据的前端 JavaScript 代码

除了设置 `Access-Control-Allow-Credentials: true` 外，还需要注意两个点：

1. 客户端请求中需设置 `withCredentials`，启用凭据发送
	```JavaScript
	// XMLHttpRequest 配置方法：
	var xhr = new XMLHttpRequest()
	xhr.withCredentials = true

	// axios 配置方法：
	axios.defaults.withCredentials = true

	// fetch 配置方法：
	fetch('url', {
		credentials: 'include'
	})
	```
2. `Access-Control-Allow-Origin` **不能**设置为 `*` （通配符），需指定具体域名

---

举个例子，假设有一个场景：

- 前端网站：`https://my-app.com`
- API 服务器：`https://api.my-app.com`
- 用户已在 API 服务器上登录，有一个会话 Cookie

服务器端配置（如 Node.JS Express）：

```JavaScript
app.use((req, res, next) => {
  // 必须指定确切的源，不能使用通配符*
  res.header('Access-Control-Allow-Origin', 'https://my-app.com');
  // 允许跨域请求携带凭据
  res.header('Access-Control-Allow-Credentials', 'true');
  // ...其他CORS头
  next();
});
```

前端代码：

```JavaScript
// 需要显式设置credentials: 'include'启用凭据发送
fetch('https://api.my-app.com/user-data', {
  credentials: 'include'  // 这会发送cookies等凭据
})
.then(response => response.json())
.then(data => console.log(data));
```

在用户访问和认证时，有如下流程：

1. 用户访问 `https://my-app.com`（前端应用）
2. 当需要登录时，用户被重定向到 `https://api.my-app.com/login`
3. 用户在 `api.my-app.com` 上完成身份认证
4. `api.my-app.com` 服务器设置认证 Cookie（如 `sessionId=abc123`）
5. 用户被重定向到 `https://my-app.com`

在后续跨域请求时，浏览器已有 `https://api.my-app.com` 的 Cookie，当 `https://my-app.com` 的 JavaScript 代码发起跨域请求到 `https://api.my-app.com` 时，使用 `credentials: include` 可以让浏览器将之前存储的 `api.my-app.com` 的 Cookie 一起发送。

### 3. JSONP

**jsonp**的原理就是利用`<script>`标签没有跨域限制，通过`<script>`标签 **`src` 属性**，发送带有 callback 参数的GET请求，服务端将接口返回数据拼凑到 callback 函数中，返回给浏览器，浏览器解析执行，从而前端拿到 callback 函数返回的数据。

原生JS实现：

```jsx
<script>
    var script = document.createElement('script');
    script.type = 'text/javascript';
    // 传参一个回调函数名给后端，方便后端返回时执行这个在前端定义的回调函数
    script.src = 'http://www.domain2.com:8080/login?user=admin&callback=handleCallback';
    document.head.appendChild(script);
    // 回调执行函数
    function handleCallback(res) {
        alert(JSON.stringify(res));
    }
 </script>
```

服务端返回如下（返回时即执行全局函数）：

```jsx
handleCallback({"success": true, "user": "admin"})
```

**JSONP的缺点：**

- 具有局限性， **仅支持get方法**
- 不安全，**可能会遭受XSS攻击**

### 4. postMessage 跨域

postMessage 是 JavaScript 提供的一种安全的跨域通信机制，允许不同源（协议、域名、端口）的页面之间进行消息传递。

#### 基本概念
- postMessage 允许一个窗口向另一个窗口发送消息，即使这些窗口来自不同源
- 它突破了浏览器的同源策略限制，但以一种受控且安全的方式

#### 语法

##### 发送消息

```JavaScript
targetWindow.postMessage(message, targetOrigin, [transfer])
```

参数说明：
- `targetWindow`：接收消息的窗口对象（如 `iframe.contentWindow`、`window.parent`、`window.opener` 等）
- `message`：要发送的数据
- `targetOrigin`：指定接收消息的窗口的源，可以是具体的 URL 或 `*`（不推荐）
- `transfer`（可选）：传输对象的数组，这些对象的所有权将被转移

##### 接收消息

接收窗口需要监听 `message` 事件：

```JavaScript
window.addEventListener('message', function (event) {
	// 验证发送者的身份
	if (event.origin !== "https://trusted-source.com") return;
	// 处理接收的消息
	console.log('收到消息', event.data)
	// event.source 是发送消息的窗口引用
	// 可以用它回复消息
	event.source.postMessage('已收到', event.origin)
})
```

> [!attention] 注意
> 为了避免 XSS 攻击，要检查消息来源，`e.origin`
#### 使用场景

1. 页面与嵌入iframe通信
2. 主窗口与弹出窗口通信
3. 页面与 Web Worker 通信
4. 跨域 API 调用
5. 单页应用程序内部通信

### 5. `nginx`

`nginx` 处理跨域有两种方法：

#### 1. 直接添加 CORS 头信息

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        # 允许的来源域名，生产环境建议具体指定域名
        add_header 'Access-Control-Allow-Origin' 'https://www.example.com';
        # 允许客户端携带认证信息
        add_header 'Access-Control-Allow-Credentials' 'true';
        # 允许的请求方法
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        # 允许的请求头
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
        
        # 处理预检请求
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain charset=UTF-8';
            add_header 'Content-Length' 0;
            return 204;
        }
    }
}
```

#### 2. 使用反向代理（更推荐）

```nginx
server {
    listen 80;
    server_name www.example.com;
    
    # 前端静态资源
    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
    
    # API 请求代理 - 不需要 CORS 配置
    location /api/ {
        proxy_pass http://api.example.com/;
        proxy_set_header Host api.example.com;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 6. WebSocket

建立持久化连接，不受同源策略影响：

```JavaScript
const socket = new WebSocket('wss://api.example.com');
socket.onopen = function() {
  socket.send('Hello Server!');
};
socket.onmessage = function(event) {
  console.log(event.data);
};
```

优点：全双工通信，实时性强

缺点：需要专门的服务器支持，协议不同于 HTTP

### 7. `document.domain`

适用于主域名相同但子域名不同的情况：

```JavaScript
// 在 a.example.com
document.domain = 'example.com';

// 在 b.example.com
document.domain = 'example.com';
// 现在两个页面可以互相访问
```

优点：简单易用

缺点：只适用于二级域名相同的情况下，且安全隐患较大

### 8. 浏览器扩展/插件

可以使用浏览器插件，来做临时的跨域测试。例如：[Allow CORS Access Control](https://chromewebstore.google.com/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf/related?hl=zh-CN)
