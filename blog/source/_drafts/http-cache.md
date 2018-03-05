---
title: http cache
comments: true
layout: false
date: 2018-02-26 14:02:52
updated: 2018-02-26 14:02:52
description: Http缓存机制及原理。
toc: true
tags:
 - http
 
---
## http报文
http报文是浏览器和服务器通信时发送及响应的数据块。浏览器向服务器请求数据，发送请求（request）报文；服务器响应浏览器请求，向浏览器返回数据，返回响应（response）报文。
request报文和response报文信息主要分为两部分：
1.http请求头（header）
 - cookie以及缓存的相关规则信息
 
2.http请求主体（body）
 - http请求传输和响应返回的数据部分