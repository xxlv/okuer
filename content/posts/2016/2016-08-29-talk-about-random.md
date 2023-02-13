---
author: X
comments: true
date: 2016-08-27 14:00:00+00:00
layout: post
title:  聊聊随机
categories: computer

---

经常遇到随机数的问题，大多数的随机目的是产生一个不太可能被预测的结果，这个结果一般具有唯一性。譬如安全的文件名之类的，生成密码重置的token。


#### 计算产生随机

那么，如何来产生一个随机数呢？以PHP为例，足够简单的方式使用如下的函数:   

``` php
$my_rand=rand(1,1000000)

```
 这个随机函数是c原生支持的，在使用rand的函数的时候，php将自动调用srand来设置随机种子。  

这个的意思是：所谓的随机结果，只是将这个seed传递给一个hash函数，返回一个“看上去足够随机”的值。这意味着，如果种子能被猜测，那么，结果将被预测。  

下面是php常见的三种生成随机数的方法:  

1. 线行同余方法 如：lcg_value
2. 梅森旋转算法 如 mt_rand
3. C语言支持的函数 如rand


#### 从物理世界来获取随机  

在linux内，维护着一个/dev/random(/dev/urandom), 你可以获得一个随机数   

``` shell
od -An -N2 -i /dev/random  
```
这个random是linux内核维护的，它从周围环境获取随机因子产生一个足够大的熵池。熵代表着系统的混乱程度，越是混乱的系统，它的随机性更大。/dev/random是怎么做的呢？  

它会收集很多毫无理由的数据，譬如在一次开机后，鼠标左键点击次数等，这些环境噪音是不可预测的（至少不能被人预测）。这是另一种产生的随机数的办法。这个随机数一般叫真随机数。它是完全不能被预测的(?).   


要产生真实的随机数，目前必须借助物理世界的噪音。   