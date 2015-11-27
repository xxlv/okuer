---
author: admin
comments: true
date: 2015-07-07 07:47:36+00:00
layout: post
slug: php%e7%9a%84coroutine
title: PHP的Coroutine
wordpress_id: 159
categories:
- PHP
post_format:
- 日志
---

摘要 PHP的Coroutine实现

**什么是Coroutine？**

**[1]**协同程序是计算机程序组件，概括子程序非抢先多任务，允许多个入口点并暂停在某些位置恢复执行。协程是非常适合用于实现更熟悉的程序组件如合作任务，异常，事件循环，迭代器，无限列表和管道。(**Coroutines** are [computer program](https://en.wikipedia.org/wiki/Computer_program) components that generalize [subroutines](https://en.wikipedia.org/wiki/Subroutine) for [nonpreemptive multitasking](https://en.wikipedia.org/wiki/Nonpreemptive_multitasking), by allowing multiple [entry points](https://en.wikipedia.org/wiki/Entry_point) for suspending and resuming execution at certain locations. Coroutines are well-suited for implementing more familiar program components such as [cooperative tasks](https://en.wikipedia.org/wiki/Cooperative_multitasking), [exceptions](https://en.wikipedia.org/wiki/Exception_handling),[event loop](https://en.wikipedia.org/wiki/Event_loop), [iterators](https://en.wikipedia.org/wiki/Iterator), [infinite lists](https://en.wikipedia.org/wiki/Lazy_evaluation) and [pipes](https://en.wikipedia.org/wiki/Pipeline_(software)).)

非阻塞/异步I/O callback风格

**
**

传统的函数是层次调用的，比如A->B->C，则C在执行中，遇到阻塞将没办法直接返回B，B和A都将处于阻塞状态（真实情况是ABC三个子程序都在抢占资源，但表现为A->B->C）。这样导致的一个后果是是可能会导致一些浪费和性能上的损失。

当使用协程的时候，这样的情况就可以避免。一个协程是一个generator，当遇到Yield关键字的时候，该方法就是协程

[![clipboard](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard_thumb.png)](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard.png)

[![clipboard[1]](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard1_thumb.png)](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard1.png)

上面的例子，使用 Coroutine的话，则不受内存限制。

协程的核心是将抢占式变成协作式。在单核CPU中可以达到最大程度的利用。不足之处是无法提升多核的利用率。但这可以使用多进行+多协程的方式进行。

在PHP中（类似Python的实现）。使用yield关键字来标志一个方法是一个generator.它本身实现了Iterator。也就是可以被foreach或while等。

[![clipboard[2]](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard2_thumb.png)](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard2.png)

上例中，gen函数是一个协程，因为其中包含了关键词yield，当调用$gen->current()的时候，将输出start。

当调用next的时候，则会一直输出直到遇到下一个yield。

$gen->send("echo");将发送到Coroutine中指令替换yeild关键词。这注意这已经不是Iterator的方法了。

IteratorAggregate接口的getIterator为foreach的提供了具体实现，例如在官方给出的demo中

[![clipboard[3]](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard3_thumb.png)](http://okuers-wordpress.stor.sinaapp.com/uploads/2015/07/clipboard3.png)

使用yield将迭代器变成协程的一个好处是，在遍历的时候，foreach负责next移动指针，但并不是将$test的数据全部载入内存，而是在next后，

yield所在的那个generator才初始化下一个next的数据，所以不管被迭代的数据量如何大，只需要满足一次迭代不会导致内存溢出即可。

[1] [https://en.wikipedia.org/wiki/Coroutine](https://en.wikipedia.org/wiki/Coroutine)

参见:[https://wiki.php.net/rfc/generators](https://wiki.php.net/rfc/generators)

参见:[http://www.cnblogs.com/Anker/p/3254269.html](http://www.cnblogs.com/Anker/p/3254269.html)
