---
layout: post
title:  "DTLSv1_listen"
date:   2022-09-12 23:00:11 +0800
categories: jekyll update
---

## DTLSv1_listen() 等待时间过长(原因分析), 超10s
### 介绍
DTLSv1_listen() 等待监听句柄上出现 ClientHello，向对方发出一个HelloVerifyRequest作为
响应，并返回0，表示当前没有客户端通过验证，它需要被再次调用以便继续监听。当客户端收到
HelloVersifyRequest的时候，会再次发送ClientHello，不过这次的Client会携带一个cookie，这时
DTLSv1_listen()会返回1，以及一个struct sockaddr 结构体。这个struct sockaddr结构体可以用来
创建一个连接到client的新的socket，以替换掉SSL会话中BIO对象里的listening socket。

### 这10s内，DTLSv1_listen() 在干什么？
1. 有无确实收到 ClientHello?

2. 有无确实发出HelloVerifyRequest?
