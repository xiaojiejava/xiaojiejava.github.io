---
layout: post
title: "跨域"
subtitle: '解决跨域请求的几种方案'
author: "xiaojiejava"
header-style: text
tags:
  - 跨域
  - 同源
  - ajax
---

### 什么是跨域

浏览器有同源策略，只要协议、域名、端口有任何一个不同，则认为是不同的域，之间的请求就是跨域请求

跨域的请求浏览器会将请求发送出去，当收到响应的时候如果没有对应的header则认为服务器不支持跨域，拒绝解析响应

### 如何解决跨域问题

1. proxy方式

	最容易想到的，在同源的服务器内后台调用跨域请求，然后再封装响应到浏览器。浏览器发起请求 -> 同源服务器(proxy)接收请求 -> 后台发起跨域请求 -> proxy封装响应 -> 浏览器接收

2. jsonp(json with padding)

	json是一种数据交换格式，而jsonp是一种非官方的跨域数据交互协议

	对于script、img标签的src属性可以跨域请求，jsonp就是利用这一点来实现的

	首先当要发起跨域请求的时候，构建一个script标签，并设置src属性值为请求的url，并指定callback参数值为当响应到达回调的js方法名，然后服务器端收到请求后处理，返回callback参数值 + 响应数据,浏览器接收到响应后会回调相应的js方法完成跨域请求

	jsonp请求有一个很大缺点是只能处理get请求,好处是支持老浏览器和不支持CORS的服务器

3. CORS(Cross-Origin Resource Sharing)

	CORS是一个W3C标准，全称是"跨域资源共享",可以解决跨域问题

	CORS需要浏览器和服务器同时支持，现在主流的浏览器都支持CORS，浏览器端可以不用考虑。它的实现原理是在请求和响应中增加一些header来支持跨域请求

	CORS分为两种请求，简单请求 和 非简单请求

	同时满足以下两个条件，就属于简单请求

		1. 请求方法是 HEAD GET POST
		2. HTTP的头信息不超过以下几种字段
			- Accept
			- Accept-Language
			- Content-Language
			- Last-Event-ID
			- Content-Type:只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
	只要不同时满足上面两个条件，就属于非简单请求

	对于简单请求，浏览器直接发出CORS请求，并在请求头中增加Origin（标明请求来自哪个源 协议 + 域名 + 端口）,如果服务器返回的头信息中没有包含Access-Control-Allow-Origin字段说明服务器不支持CORS，浏览器直接抛出一个错误，被ajax请求的onerror回调函数捕获

	如果Origin指定的域名在许可范围内，服务器返回响应，会多出几个头信息字段
	
		1. Access-Control-Allow-Origin: xxx 必须字段，表示支持跨域请求的源
		2. Access-Control-Allow-Credential: true 可选字段，表示是否支持浏览器发送cookie
		3. Access-Control-Expose-Headers: xxx 浏览器只能拿到响应header中的6个字段,Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pargma，如果想拿其它字段，就必须在这里指定
		4. Content-Type: xxx

	如果浏览器端想发送cookie等信息，必须在ajax请求中打开withCredentials属性，即xhr.withCredentials = true

	对于非简单请求，例如请求方式是PUT或DELETE，或者Content-Type=application/json在正式发送CORS请求之前，需要先发送一个预检请求(preflight)

	浏览器先询问服务器，当前网页所在的域名是否在服务器允许跨域的许可名单内，以及可以使用那些HTTP动词和头信息，得到确认后在发送请求

	预检请求的方法是OPTIONS，会携带两个特殊的头信息.
	
		1. Access-Control-Request-Method 必须的，指定请求方法
		2. Access-Control-Request-Headers 逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息

	服务器收到OPTIONS请求后，响应头有如下信息:
	
		1. Access-Control-Allow-Methods 必须字段，逗号分隔的字符串，返回所有支持的请求方法
		2. Access-Control-Allow-Headers 逗号分隔的字符串，表示服务器支持的所有头信息字段
		3. Access-Control-Allow-Credentials
		4. Access-Control-Max-Age 可选字段，指定本次预检请求的有效期

	浏览器收到预检请求的确认信息后，发起正常的CORS请求，这时候就跟普通请求一样，带有Origin，响应头带有Access-Control-Allow-Origin头

	Origin头是浏览器自动添加的




