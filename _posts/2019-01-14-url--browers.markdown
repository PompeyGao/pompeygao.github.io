---
layout:     post
title:      "从输入URL到页面加载完成的过程中都发生了什么事情？"
subtitle:   ""
date:       2019-01-14
author:     "pompeygao"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - 前端开发
---



1. 输入地址
2. 浏览器查找域名的 IP 地址这一步包括 DNS 具体的查找过程，包括：浏览器缓存->系统缓存->路由器缓存...
3. 浏览器向 web 服务器发送一个 HTTP 请求
4. 服务器的永久重定向响应（从 http://example.com 到 http://www.example.com）
5. 浏览器跟踪重定向地址
6. 服务器处理请求
7. 服务器返回一个 HTTP 响应
8. 浏览器渲染 HTML
9. 浏览器发送请求获取嵌入在 HTML 中的资源（如图片、音频、视频、CSS、JS等等）
10. 浏览器发送异步请求