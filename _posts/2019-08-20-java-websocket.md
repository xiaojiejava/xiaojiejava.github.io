---
layout: post
title: "WebSocket入门介绍"
subtitle: '介绍HTTP/1.1 HTTP/2.0，用WebSocket实现服务器主动向客户端发送消息，使得浏览器具备了实时双向通信的能力'
author: "xiaojiejava"
header-style: text
tags:
  - HTTP/1.1
  - HTTP/2.0
  - WebSocket
  - 双向通信
  - 服务器推送
---

这边文章主要介绍WebSocket，这里先介绍下HTTP协议再讲WebSocket

### HTTP协议

HTTP(HyperText Protocol)，超文本传输协议，是一个基于TCP实现的应用层协议

HTTP的报文有两种类型：请求和响应，其报文格式分别为：

请求报文格式

	- 请求方法 URL HTTP版本号
	- 请求首部字段（可选）
	- 空行
	- body

响应报文格式

	- HTTP版本号 返回码 返回码描述
	- 响应首部字段（可选）
	- 空行
	- body

HTTP/1.0的劣势：

1. 连接无法复用，对于每一个HTTP请求都对应一个TCP请求，每次都要经历三次握手和慢启动，对于常规的一个页面嵌入较小的css/js/jpg，也要单独建立一个TCP连接，效率低。

2. HTTP Head of line Blocking，由于每个浏览器对同一个域名都有并发数限制，则必须等待上一个请求/响应结束后下一个请求才能发出，可能导致后续高优先级请求阻塞。

HTTP/1.1解决方案：

1. 支持长连接，设置Connection: Keep-Alive，开启持久连接，实现TCP连接复用。

2. HTTP pipelining(管道)将多个HTTP请求一次发送到一个TCP连接上，实现TCP连接复用，但是浏览器响应也是按顺序处理的，且这种方式有较大的限制 & 浏览器/服务器不一定支持

HTTP/1.1劣势：

1. 虽然支持HTTP pipelining，但是响应还是按照顺序的，没有实现真正的连接复用。

2. 臃肿的HTTP头部，每次请求都要携带重复的header，如果有cookie对性能影响更大。

3. 不支持请求优先级

4. 不支持服务器推送，客户端需要资源必须主动发起请求获取

HTTP/2.0解决方案：

1. 二进制分帧层，它定义了如何封装HTTP消息并在客户端和服务器之间传输。HTTP语义（包括各种动词、方法、标头）都不受影响，不同的是传输期间对它们的编码方式变了。HTTP/1.x以换行符作为纯文本的分隔符，
而HTTP/2将所有传输的信息分割为更小的消息和帧，并采用二进制格式对它们编码。实现了一个TCP连接多个请求，且请求的顺序互不影响，同一个域名只需要用一个TCP连接处理所有HTTP请求。

2. HTTP头部压缩，采用HPACK压缩头部。

3. 支持优先级设置，需要客户端/服务器支持。

4. 支持服务器推送，例如如果请求是一个html则将里面的css/js/jpg等随html响应一块推送给客户端，减少客户端再次请求

设想一个这样的场景，页面有一个二维码和操作按钮，当用户扫描二维码成功后操作按钮才可点击，问题是如何监听这个二维码是否扫描成功了?

1. 定时轮询，利用setInterval或setTimeout定时向后台发ajax请求，如果已经探测到二维码扫描则停止轮询。
	
	优势：实现简单
	劣势：
		- 浪费服务器资源，大部分的请求其实都是无效的，但是服务器每次都得处理请求
		- 响应速度慢，受限于定时频率，时效性低
		- 每次都要发送完整的HTTP请求信息，比如header

2. 定时长轮询，每次ajax请求当探测到二维码未扫描的时候不会立即返回，而是会阻塞直到有新消息的时候在返回响应。这种方式又称为comet。

3. WebSocket，可以实现客户端和服务端的双向通信，它跟HTTP协议没关系，是一个新的应用层协议，只是借助HTTP实现协议升级。
	
	优势和劣势完全和定时轮询相反

### WebSocket原理实现

首先客户端这边发起协议升级请求：

- GET /websocket HTTP/1.1
- Host: xxx
- Connection: Upgrade --- 表示要升级协议
- Upgrade: websocket  --- 表示要升级到的协议是websocket
- Sec-WebSocket-Version: 13 --- 表示websocket版本，如果服务器不支持该版本，需要返回一个Sec-WebSocket-Version header，里面包含服务器支持的版本
- Sec-webSocket-key: xxx --- 这是一个Base64 encode值，这个是浏览器随机生成的

服务器响应如下:

- HTTP/1.1 101 Switching Protocols -- 表示协议升级成功
- Connection: Upgrade
- Upgrade: WebSocket
- Sec-WebSocket-Accept: xxx  ---根据请求header Sec-WebSocket-key计算得到，规则为toBase64(sha1(Sec-WebSocket-Key + 固定串))

### 使用WebSocket中遇到的坑

1. session共享的问题

	如果服务器是集群，则客户端可能与任意一台服务器建立持久连接，可能和处理后台消息的服务器不是同一台，则消息推送无法实现。

	- 解决方案是通过消息队列来实现，服务器监听消息队列，当某台服务器收到请求处理成功消息后将其发送给消息队列，各个服务器拉取到消息判断是否需要推送，如需则推送否则忽略消息

2. 部分浏览器不支持WebSocket或者WebSocket连接失败

	如果连接失败，降级到定时轮询方案

3. 代理服务器例如nginx有超时设置，可能在超时时间内监听并没有结束
	
	- 调大代理服务器超时时间
	- 客户端定时发送心跳包连接保活

4. 服务上线持久连接断掉问题

	Graceful Shutdown，服务器从服务发现中自动移除 -> 主动通知浏览器重连 -> 重新连接到新的服务器

### WebSocket和HTTP/2.0服务器推送的区别

- HTTP/2.0的服务器推送主要目的是减少HTTP请求 & 提升页面加载速度，而且客户端并没有一个丰富的API能操作服务器推送过来的数据
- WebSocket客户端和服务器有丰富的API可用，目的是解决全双工通信的问题
- 未来二者可以融合，WebSocket基于HTTP/2.0来实现



