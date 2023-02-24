---
title: "Go No Copy"
date: 2023-02-24T10:09:05+08:00
---

## 什么是no copy?

Lock & Unlock 代表对一种临界资源的保护。go 提供的 `vet` 工具可以针对`copylocks`进行检查。

但是如`waitGroup` 等同样需要配对操作的组件并没有更好的办法来阻止开发者。

因此在 [#8005](https://github.com/golang/go/issues/8005#issuecomment-190753527)中 采用了 `noCopy` 的struct.

需要禁止copy就可以include 这个`struct` .


## 为什么不能？

设想,如果允许copy lock ，会发生什么?

![lock](/d2/nocopy.png)


如上会产生意外的情况，G1的lock被copy到G2中，被意外的提前`unlock` 了。会导致 G3在 G1 还没有`unlock` 的时候，获得写数据的权限。

为了规避上述问题，采用了 `noCopy` 方案。




