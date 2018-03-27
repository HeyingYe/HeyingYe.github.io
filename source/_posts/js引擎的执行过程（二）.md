---
title: js引擎的执行过程（二）
comments: true
date: 2018-03-26 10:17:30
updated: 2018-03-26 10:17:30
description: 上文我们分析了js引擎执行过程的语法分析和预编译阶段，本文将会具体分析第三个阶段--执行阶段。
toc: true
tags:
 - javascript

---
## 概述
js引擎执行过程主要分为三个阶段，分别是语法分析，预编译和执行阶段，上篇文章我们介绍了语法分析和预编译阶段，我们先做个简单概括，如下：
- **语法分析**： 分别对加载完成的代码块进行语法检验，语法正确则进入预编译阶段；不正确则停止该代码块的执行，查找下一个代码块并进行加载，加载完成再次进入语法分析阶段

- **预编译**：通过语法分析阶段后，进入预编译阶段，创建变量对象（创建arguments对象（函数运行环境下），函数声明提前解析，变量声明提升），确定作用域链以及this指向。

如还有疑问回头看看上篇文章[js引擎的执行过程](https://heyingye.github.io/2018/03/19/js%E5%BC%95%E6%93%8E%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B/)。
<br>
本文主要分析js引擎执行的第三个阶段--**执行阶段**，在分析之前我们先思考以下两个问题：
>js是单线程的，为了避免代码解析阻塞使用了异步执行，那么它的异步执行机制是怎么样的？

通过事件循环（Event Loop），理解了事件循环的原理就理解了js的执行机制，本文主要介绍。

>js是单线程的，那么参与js执行过程的就只有一个吗？

不是的，会有四个线程参与该过程，但是永远只有JS引擎线程在执行JS脚本代码，其他三个线程只协助，不参与代码解析与执行。参与js执行过程的线程分别是：
 - **JS引擎线程**： 也称为JS内核，负责解析执行Javascript脚本程序（例如V8引擎）

 - **事件触发线程**： 归属于浏览器内核进程，不受JS引擎线程控制。主要用于控制事件（例如鼠标，键盘等事件），当事件被触发时候，事件触发线程就会把该事件的处理函数推进**事件队列**，等待JS引擎线程执行

- **定时器触发线程**：主要控制计时器setInterval和延时器setTimeout，用于定时器的计时，计时完毕满足定时器的触发条件，则将定时器的处理函数推进事件队列中，等待JS引擎线程执行。
注：W3C在HTML标准中规定setTimeout低于4ms的时间间隔算为4ms。

- **HTTP异步请求线程**：通过XMLHttpRequest连接后，通过浏览器新开一个线程，监控readyState状态变更时，如果设置了该状态的回调函数，则将该状态的处理函数推进事件队列中，等待JS引擎线程执行。
注：浏览器对通一域名请求的并发连接数是有限制的，Chrome和Firefox限制数为6个，ie8则为10个。

总结：永远只有JS引擎线程在执行JS脚本程序，其他三个线程只负责将满足触发条件的处理函数推进事件队列，等待JS引擎线程执行。

## 执行阶段
我们还是先分析一个典型的例子（来自[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)）：
```
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```
这里我直接对例子进行简单分析，暂不解释概念和原理，概念和原理将会在下面具体讲解，以上代码按任务类型进行划分，如下：
1. 宏任务（macro-task），宏任务又按执行顺序分为同步任务和异步任务
    - 同步任务
      ```
        console.log('script start');

        console.log('script end');
      ```
    - 异步任务
    ```
        setTimeout(function() {
          console.log('setTimeout');
        }, 0);
    ```

2. 微任务（micro-task）
```

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

```
在JS引擎执行过程中，进入执行阶段后，代码的执行顺序如下：
```
宏任务(同步任务) --> 微任务 --> 宏任务(异步任务)
```
输出结果为：
```
script start
script end
promise1
promise2
setTimeout
```

相信很多人都不理解为什么会出现这样的结果，那么我们接下来对上面例子进行详细分析。

### 宏任务
宏任务（macro-task）可分为**同步任务**和**异步任务**：
- 同步任务指的是在JS引擎主线程上按顺序执行的任务，只有前一个任务执行完毕后，才能执行后一个任务，形成一个执行栈（函数调用栈）。

- 异步任务指的是不直接进入JS引擎主线程，而是满足触发条件时，相关的线程将该异步任务推进**任务队列(task queue)**，等待JS引擎主线程空闲时，读取执行的任务，例如异步Ajax，DOM事件，setTimeout等。

理解宏任务中同步任务和异步任务的执行顺序，那么就相当于理解了JS异步执行机制--**事件循环（Event Loop）**。



#### 事件循环
事件循环可以理解成由三部分组成，分别是：
- 主线程执行栈

- 异步任务等待触发

- 任务队列

**任务队列(task queue)就是以队列的数据结构对事件任务进行管理，特点是先进先出，后进后出**。
<br>
这里直接引用一张著名的图片(参考自Philip Roberts的演讲《Help, I'm stuck in an event-loop》)，帮助我们理解，如下：
![Event Loop](img/Event Loop.jpg)

在JS引擎主线程执行过程中：
- 首先执行宏任务的同步任务，在主线程上形成一个执行栈，可理解为函数调用栈；

- 当执行栈中的函数调用到一些异步执行的API（例如异步Ajax，DOM事件，setTimeout等API），则会开启对应的线程（Http异步请求线程，事件触发线程和定时器触发线程）进行监控和控制

- 当异步任务的事件满足触发条件时，对应的线程则会把该事件的处理函数推进任务队列(task queue)中，等待主线程读取执行

- 当JS引擎主线程上的任务执行完毕，则会读取任务队列中的事件，将任务队列中的事件任务推进主线程中，按任务队列顺序执行

- 当JS引擎主线程上的任务执行完毕后，则会再次读取任务队列中的事件任务，如此循环，这就是事件循环（Event Loop）

上面的例子中宏任务的代码部分是：
```
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

console.log('script end');
```
代码执行过程为：
1. JS引擎主线程按代码顺序执行，当执行到``console.log('script start');``，JS引擎主线程认为该任务是同步任务，所以立刻执行输出script start，然后继续执行；

2. JS引擎主线程执行到``setTimeout(function() {
  console.log('setTimeout');
}, 0);``，JS引擎主线程认为setTimeout是异步任务API，则向浏览器内核进程申请开启定时器线程进行计时和控制该setTimeout任务，由于W3C在HTML标准中规定setTimeout低于4ms的时间间隔算为4ms，那么当计时到4ms时，定时器线程就把该回调处理函数推进任务队列中，等待主线程执行

3. JS引擎主线程执行到``console.log('script end');``，JS引擎主线程认为该任务是同步任务，所以立刻执行输出script end

4. JS引擎主线程上的任务执行完毕（输出script start和script end）后，主线程空闲，则开始读取任务队列中的事件任务，推进主线程中，按任务队列顺序执行，输出setTimeout

理解事件循环后，我们做一些拓展性的思考：
>我们都知道setTimeout和setInterval是异步任务的定时器，需要添加到任务队列等待主线程执行，那么使用setTimeout模拟实现setInterval，会有区别吗？

答案是有区别的，我们不妨思考一下：
- setTimeout实现setInterval只能通过递归调用

- setTimeout是在到了指定时间的时候就把事件推到任务队列中，只有当在任务队列中的setTimeout事件被主线程执行后，才会继续再次在到了指定时间的时候把事件推到任务队列，那么setTimeout的事件执行肯定比指定的时间要久，具体相差多少跟代码执行时间有关

- setInterval则是每次都精确的隔一段时间就向任务队列推入一个事件，无论上一个setInterval事件是否已经执行，所以有可能存在setInterval的事件累积，导致setInterval的代码连续执行多次


## 微任务
微任务是在es6和node环境中执行


## 参考文献
- [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)