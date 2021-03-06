---
author: admin
comments: true
date: 2015-12-21 08:05:39+00:00
layout: post
title: 为什么计算机越来越复杂
wordpress_id: 81
categories:
- 杂文
---

从编程语言的进化路线来看，似乎能发现一些有趣的事情。计算机通过０和１将数字信息转化为电信号控制硬件的运作，所有的一切都建立在这个基础上。计算机最擅长的是重复，而且仅仅只有两种状态间重复。这些简单的规则构成了复杂无比的系统。我也抱着同样的理由深信这个世界的底层是有一系列简单的规则构成的。然而，你无法通过直接操作０和１来控制计算机，虽然这是一种可能性，但现实是，没有人愿意这样做。在计算机迅速发展的历程中，跟计算机交流的语言越来越多，汇编是对机器码（０和１）的抽象。它使用英文单词赋予语言以明确的语义。实际上，它所做的并不仅仅如此，它同样隐藏了大量的重复。用一种指令来代表一系列操作产生的结果集。这种思维在计算机的世界变成一种哲学。不久后，更复杂的需要导致理解汇编变得异常困难。更高级的语言诞生了，在这些语言中，C是一颗耀眼的明星。它的语义更加明确，规范，它隐藏了底层的许许多多细节。用C来创建超大型的软件有了可能，window和unix*是它最伟大的代表作。  

C语言是一种艺术，这是一种激动人心的创造。就像是在魔法森林找到的咒语一样。这些咒语被创造出各种奇妙的东西，图形渲染，音频处理。计算机内部拥有无限中状态等待被开启。  

在C语言的内部，你仍旧能发现无数对重复的隐藏，函数封装了重复的细节，宏被赋予更伟大的魔力。函数群一种有执行力的软件。软件将功能通过API暴漏出去。抽象时刻在进行，所有的东西都被包装在集装箱里，被打包成更大的集装箱。  

对于人类而言最痛苦的是不断的重复（然而我们也一直在重复），在我们有限的生命里，重复是多么可怕的事情，然后对于机器来讲，重复是它们最擅长也乐此不疲的事情。我们刻意隐藏重复，用一种更加乐观的态度，用天气的不同，日历的不同，地理位置的不同来告诉自己，我们不是机器，没有重复自己。   

很多更高级的语言由此诞生，它们隐藏了更多的重复（这也是它们慢的原因之一），越是高级的语言速度越慢，隐藏的细节越多，简单的逻辑背后的重复越多，这些重复将表现为磁头在扇区上的物理移动（至少目前的计算机物理架构是这样的）、一次次的内存访问、重复GC、对空间的申请、释放和锁。   

为什么事情越来越复杂，为什么普通人无法理解计算机，计算机逃脱意味着什么？   

我们很容易跟小孩子讲清楚1+1这样的加法规则，然后加法重复之后。   

如2+3=5，他很快就能理解，因为这些法则只是简单的抽象。但变成2*3=6这个过程，你需要解释（2+2+2）或者(3+3)，同一个重复被抽象之后有了多层含义，但它们都是多个正确的解。  

又如2**4（代表2*2*2*2）这样的重复展开后其实是

``` shell
    2+2+2+2+2+2+2+2
```

。它的语义是：重复４次２的累积过程。要知道2这个数其实是我们定义的一个阿拉伯符号而已。  

２代表着1+1这个表达式的值。所以
``` shell
    2+2 +2+2 +2+2+2+2
```

还可以继续展开变成
``` shell
    1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1
```
减法也可以看做是一种累计。不过它带来的副作用是削弱。  

甚至１（实际上，０－９任何一个数都是）本身也是一种抽象，这是我们的世界的基础，代表着存在本身.

计算机正是从这些很简单的规则不断的抽象，隐藏自身，和进化的。所以，越是高级的抽象就越难以描述，为此我们制造了大量的术语和符号来表达这种东西。将重复隐藏的代价就是使用命名它。  

这样它就变成一个全新而陌生的东西。  
