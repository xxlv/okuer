---
title: "GPM调度模型解析"
date: 2023-02-14T15:16:24+08:00
---

## 什么是GPM?
G-P-M 模型是 go 语言最重要的调度模型。 

简单来说,开发者编写的任务包装成G交给P,M在P的G队列上获取G进行计算。

### KSE内核调度

> KSE (Kernel Scheduling Entity) 是FreeBSD内核中用于调度线程的实体。KSE内核调度是指FreeBSD内核根据KSE的状态、优先级和其他因素来决定在何时执行某个KSE上的线程。KSE内核调度采用多级反馈队列调度算法，并考虑了线程的优先级、I/O等待、CPU时间片使用情况等因素，以提高系统的性能和响应能力。

### 线程与进程
操作系统的最小调度单元是线程，线程可以共享进程内的资源.

#### 操作系统的调度模型
一切都是资源，无论计算还是存储，操作系统来负责协调

##### 计算机结构
- SMP
>对称多处理器（Symmetric Multiprocessing，SMP）是一种计算机体系结构，其中多个处理器共享同一物理内存和系统总线，可以同时执行多个线程或进程。


- NUMA
>非一致性存储（Non-Uniform Memory Access，NUMA）是一种多处理器计算机体系结构，在该体系结构中，每个处理器与本地内存模块相连，但可以通过互连网络访问其他处理器的内存模块。由于访问远程内存可能需要更长的时间，因此 NUMA 系统中的内存访问时间不是均匀的。

#### OS线程模型
- 内核态
> 操作系统管理,绑定一个KSE，可以看作就是一个KSE，KSE用来分配CPU

- 用户态
> 用户态的线程需要映射到内核态线程，属于用户空间上虚拟的线程，最终还是要通过KSE来分配CPU


#### 调度的实现

- 用户级 1:N

![label](/atts/2.png)

- 内核级 1:1

![label](/atts/3.png)


- 两级线程模型 N:N

![label](/atts/4.png)


##### 为什么这么设计？

计算一般分为 **CPU密集型** 和 **IO密集型** 

CPU密集的计算往往依赖CPU,因此直接KSE进度调度CPU速度更快。一些需要IO的，往往时间比较久，如果长时间占据CPU会导致空闲等待。
因此不同的线程模型应对不同的场景。 

比如一个在一个4C的计算机上进行CPU密集运算，通过4个KSE进行调度的性能更高，因为不需要切换上下文。
而一个处理数据库链接的，如果还按照这个模型，则最多处理4个链接,因此需要在执行IO的时候快速的让出CPU. 此种条件下，选择1:N或N:N的模型可以获得更高的并发。

{{< notice note >}} 用户级的线程 1:N 注意无法充分利用多核CPU.. {{< /notice >}}

## Go的调度模型

### GM模型
Go语言引入了 `Goroutine` 和KSE绑定 实现两级线程模型。用户以非常低的成本创建大量的 `G`, `M` 调度 `G` 

简单的从G-M 模型天然带有缺陷：

>  下面的解释 参考 [draveness](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/)
- 调度器和锁是全局资源，所有的调度状态都是中心化存储的，锁竞争问题严重
- 线程需要经常互相传递可运行的 Goroutine，引入了大量的延迟
- 每个线程都需要处理内存缓存，导致大量的内存占用并影响数据局部性
- 系统调用频繁阻塞和解除阻塞正在运行的线程，增加了额外开销

### P的引入
P是一个逻辑处理器，它持有一个绑定在其上的 `G` queue。 它拥有自己的局部队列，因此可以避免锁竞争。
同时它还可以通过工作窃取算法，以充分的利用CPU。一个`P`会在一个`M` 上进行计算。

P是一个中间层, 拥有M 调度所需要的上下文,一个P必须通过M调度，M不直接调度G。这种设计使得可以创建的G成本非常廉价。目前go语言创建一个G仅仅需要2k。
而如Java 创建的传统的线程成本可能在2M。


{{< notice info >}}Java19 提供了[虚拟线程](https://docs.oracle.com/en/java/javase/19/core/virtual-threads.html#GUID-7F5DA570-4B24-4CF6-899C-0424464B6032),以更廉价的方式创建线程 {{< /notice >}}


![label](/atts/1.png)



## 参考

- https://docs.oracle.com/en/java/javase/19/core/virtual-threads.html#GUID-DC4306FC-D6C1-4BCC-AECE-48C32C1A8DAA
- https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651453238&idx=1&sn=e5751f48ff880a5545819f800c799370&chksm=80bb29c4b7cca0d2d5d9cab4f4dd7e28ba8f85c5977ef20cce5aeade00e5ec1fe228f8b950e9&scene=27
- https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/