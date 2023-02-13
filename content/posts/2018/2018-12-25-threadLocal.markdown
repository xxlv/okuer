---
author: ghost
comments: true
date: 2018-12-25 08:05:39+00:00
layout: post
title: 谈谈ThreadLocal
categories:
- Tech
---

-------------------

#### 摘要

ThreadLocal被设计为在线程内部共享的变量。

并发操作带来极大的性能优势，但同时在数据的一致性上也带来了挑战，当我们在线程内部要传递一个变量，如果将变量设置为局部的，当然没有任何并发问题，但是在同一个线程里，其他的方法就不能共享该变量。
如果设置为全局类变量，又导致多个线程会共享，如果解决这个问题呢？

ThreadLocal就是为了这个目的诞生的，它提供了一种具有特殊能力的“线程私有变量”，在当前线程中，可以随意共享并且随意使用，在其他线程中，它是完全独立的。


#### 使用场景

线程内部的共享的变量在许多场景下非常有用，比如经典的web 服务，一次用户的请求，会分配一个线程去处理，使用ThreadLocal可以与一个sessionId绑定，这样的话，一个线程就处理一个用户的请求。  


#### 实现原理

ThreadLocal需要被设计为线程私有的，要完成这个目标，我们一般会想到借助于HashMap，使用K->V 的形式。每个线程的维护一个HashMap。

比如这样设计

``` java
threadMap=getMapByThread(currentThread);

threadMap.put(ThreadLocal.KEY,threadLocal.VALUE)

```
这样，让每个线程都能获取到自己的map。就可以将threadLocal在线程中隔离出来。

在JDK的实现中，也是采用了类似的原理，每个ThreadLocal都维护着一个map。但是区别是，这个Map是它自己实现的ThreadLocalMap而非Hashmap.

map的key就是ThreadLocal。


#### 为什么要自己实现ThreadLocalMap？


要回答这个问题，就要考虑如下的情况:

``` java

threadLocal=null;

```

此次，我们的预期是threadLocal被设置为null。在GC的时候，需要回收内存。但是，注意，在ThreadLocal内部的map中，ThreadLocal被作为key。

ThreadLocal是有引用的，所以，在这里 threadLocal并不会被GC掉。这样带来的问题就是，线程尤其是常驻线程的大量threadLocal都不会被释放，很容易引起OOM。


在解决这个问题上，我们看到JDK是通过weakReference来解决的。所以自然不能用Hashmap。只能用自己实现的ThreadLocalMap来解决。


但是，对ThreadLocalMap的key做弱引用，还是导致map存在一些 NULL->V 的数据。这样的脏数据也会持续占据内存。 JDK只能在set、remove的时候，将Key为NULL的数据删除。这样做显然是无奈之举。因为，你不能指望一个用户会如你预期的调用这些方法。如果用户不调用这些方法，仍旧会存在OOM问题。


#### 小结

看到 ThreadLocal的实现其实并不完美，甚至有点丑，但ThreadLocal已经完全满足工业需求。在技术上的实现并不总是有完美的解决方案，我们看到每个通过set、remove后，这个方法总是会处理一些额外的事情，这些事情用户也许并不知情。用户期望要一个杯子，但是服务员给了一个杯子，顺便倒掉了里面剩余的水。总的而言，用户得到了他想要的，而服务员也很好的规避了用户的投诉。虽然，这一些都是在用户也许不知情的情况下发生的。