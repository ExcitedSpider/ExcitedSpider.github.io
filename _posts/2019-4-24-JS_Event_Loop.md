---
layout: post
title: "JS Event Loop"
date: 2019-4-24
excerpt: "JS Event Loop"
tags: [JavaScript]
comments: false
---

# JS Event Loop

说的不对请轻拍。

首先，JS是单线程非阻塞脚本语言，这是JS出生时就决定了，也是因为这个特性让JS完美契合浏览器环境需要。而JS的并发模型基于event loop实现。

一个JS引擎包含了一个待处理的消息队列。每一个消息都关联着一个用以处理这个消息的函数。

当JS引擎空闲时，就会去消息队列取消息。而真正的定时是由宿主环境完成的。比如setTimeout就是使用宿主的定时器，当时间到达时，宿主将事件插入到消息队列中；Ajax就是网络请求到达时，宿主将事件插入到消息队列中。

之所以称为Event Loop，是因为实现类似这种循环（然而是异步的，这种循环伪代码是同步的）：

```js
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

## 特性

- 执行至完成

  每一个消息完整的执行后，其它消息才会被执行，而不用考虑被抢占。但这样缺点就是在于当一个消息需要太长时间才能处理完毕时，Web应用就无法处理用户的交互。

- 添加消息

  在浏览器中，添加消息主要是setTimeout定时器事件，DOM事件

- 零延迟

  零延迟意为着就算setTimeout 0也不会立即执行，而是立即进入消息队列。什么时候被执行取决于前面有多少事件在排队和同步代码要执行多久。

## 实现

1. JS引擎执行调用栈中的代码，调用栈可能是由于同步代码引起的也可能是异步代码引起的。
2. JS执行至调用栈清空，即当前执行函数完毕，从消息队列队首取出任务，放入调用栈中执行。
3. 若栈中的代码产生了消息，则把消息放入消息队列队尾等待执行。
4. 重复执行。

## 两级消息队列

- 宏队列**macrotask**

  常说的消息队列。包括DOM消息，setTimeout，setInterval，等等。

- 微队列**microtask**

  ES6和Node才有的的机制。比如Promise，process.nextTick(Node独有)，等等。特点是会抢占宏队列。即微队列优先级大于宏队列，如果在微队列消息的回调中产生了微队列消息，直接插入到微队列中，类似操作系统的多级FCFS队列。 