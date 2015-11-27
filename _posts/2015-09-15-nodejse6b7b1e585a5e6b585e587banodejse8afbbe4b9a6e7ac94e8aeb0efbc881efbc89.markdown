---
author: admin
comments: true
date: 2015-09-15 12:49:02+00:00
layout: post
slug: nodejs%e6%b7%b1%e5%85%a5%e6%b5%85%e5%87%banodejs%e8%af%bb%e4%b9%a6%e7%ac%94%e8%ae%b0%ef%bc%881%ef%bc%89
title: 深入浅出NodeJS读书笔记（1）
wordpress_id: 337
categories:
- NodeJS
- 读书笔记
---

最近在读《深入浅出NodeJS》。下面记录一些容易遗忘的知识。

相关名词：事件循环，非阻塞IO，观察者，I/O线程池,请求对象(XXReqWrap)，回调函数

在NodeJS的底层维护这一个线程池（libuv维护），libuv是NodeJS为了处理不同平台(windows + *uix)异步抽象出的一层，对不同平台的异步实现有不同的方案。在windows下采用IOCP来实现异步，而在linux下用node自己维护的线程池。

在异步编程中，执行完毕的逻辑并不会立即返回业务需要的数据(传统同步编程中的“返回值”)，回调函数在NodeJS中占有重要的地位，注意它并不是显式的调用的（即不是由编写者调用的）。

异步IO请求会开始一个专门处理IO的线程，该线程独立于NodeJS的执行线程，所以并不会阻塞NodeJS代码的后续执行。在线程中执行完毕IO操作后，通过将结果传递给I/O观察者(在事件循环里)。I/O观察者获得到可用的请求对象，并取出回调执行。

事件循环是一个类似while(1)的事件监测过程，在这里，有许多观察者，譬如IO观察者等（观察者分优先级）。事件循环体内，每一次循环都遍历观察者们，如有事件，则获得并执行相应的回调。

注意：在每一轮循环中检查中，idle观察者>IO观察者>check观察者。每一次循环都是一次Tick。

下面是几个有趣的函数：

setImmediate函数（属于check观察者），跟js中的setTimeout函数类似，延迟执行，不过，这里的延迟是一个循环，也就是执行完毕该函数后，进入下一个事件循环。

process.nextTick（属于idle观察者），将回调函数放入队列，在下一轮Tick中取出。

参看：[http://www.infoq.com/cn/articles/nodejs-asynchronous-io/](http://www.infoq.com/cn/articles/nodejs-asynchronous-io/)
