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
javascript是单线程语言
>在浏览器中一个页面永远只有一个线程在执行js脚本代码（在不主动开启新线程的情况下）。

javascript是单线程语言,但是代码解析却十分的快速，不会发生解析阻塞。
>javascript是异步执行的，通过事件循环（Event Loop）的方式实现。

下面我们将分别通过两个例子来分析**js引擎执行过程**以及**事件循环（Event Loop）**。

## js引擎执行过程
```
<script>
    console.log(fun)

    console.log(person)
</script>

<script>
    console.log(person)

    console.log(fun)

    var person = "Eric";

    console.log(person)

    function fun(){
        console.log(person)
        var person = "Tom";
        console.log(person)
    }

    fun()
</script>
```

我们可以先分析上面的代码，按自己的理解分析输出的顺序是什么，然后在浏览器执行一次，结果一样的话，那么代表你已经对js的执行过程有了一定的理解，如果不是，则代表我们对执行过程还存在模糊或者概念不清晰等的问题。结果我们不在这里进行讨论，我们直接分析js的执行过程，相信在理解该过程后我们就不难得出结果的，js执行过程分为三个阶段：
1. 词法分析

2. 预编译阶段

3. 执行阶段

（注：浏览器首先按顺序加载由``<script>``标签分割的js代码块，加载js代码块完毕后，立刻进入以上三个阶段，然后再按顺序查找下一个代码块，再继续执行以上三个阶段，无论是外部脚本文件还是内部脚本代码块，都是一样的原理，并且代码块之间可以共同访问全局变量。）

### 词法分析
加载js脚本代码块完毕了，会首先进入词法分析阶段。该阶段主要作用是：

 >分析该js脚本代码块的语法是否正确，如果出现不正确，则向外抛出一个语法错误（SyntaxError），停止该js代码块的执行，然后继续查找并加载下一个代码块

![syntax](img/syntax.jpg)
### 预编译阶段

