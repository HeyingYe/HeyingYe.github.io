---
title: js引擎运行机制
comments: true
date: 2018-03-19 14:19:07
updated: 2018-03-19 14:19:07
description: js是一种非常灵活的语言，理解js引擎的运行机制对我们理解javasscript非常的重要，本文通过分析js引擎的执行顺序以及事件循环，在一定程度上加深我们对js的理解。
toc: true
tags:
 - javascript
---

## 概述
js脚本代码的执行和解析是由js引擎负责的，而js引擎是单线程，意思就是说在浏览器中一个页面永远只有一个线程在执行js脚本代码（在不主动开启新线程的情况下）。下面我们将分别通过两个例子来分析**js引擎的执行顺序**以及**事件循环（Event Loop）**。

## js引擎的执行顺序
```
console.log(name)

console.log(fun)

var name = "Eric";

console.log(name)

function fun(){
    console.log(name)
    var name = "Tom";
    console.log(name)
}

fun()
```