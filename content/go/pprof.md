---
title: "Go性能调优工具"
date: 2023-02-16T11:06:01+08:00
---

# Tools 
- pprof 
- Bench 

## Pprof 

### 使用

导入
```go
import _ "net/http/pprof"

```

```go
go func() {
	log.Println(http.ListenAndServe("localhost:6060", nil))
}()
```

采集
```go 
go tool pprof http://localhost:6060/debug/pprof/heap
```
``` shell
Fetching profile over HTTP from http://127.0.0.1:6060/debug/pprof/heap
Saved profile in /Users/lvxiang/pprof/pprof.alloc_objects.alloc_space.inuse_objects.inuse_space.004.pb.gz
Type: inuse_space
Time: Feb 16, 2023 at 11:20am (CST)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)

```

参考命令，比如保存成为`png`  生成图片如下

![L](/atts/profile001.png)


### 特定场景

- 采集5s的trace 
``` shell
 curl -o trace.out http://localhost:6060/debug/pprof/trace\?seconds\=5
```

> 查看应用程序的运行时间和延迟，以及在代码执行期间发生的事件。
了解 goroutine 的创建和销毁时间，以及它们在执行期间的状态和行为。
分析应用程序的并发性能，并发执行的 goroutine 数量、互斥锁的竞争情况等。
进一步了解内存使用情况，查看内存分配和释放的模式和数量。
使用

```shell
go tool trace trace.out
```


查看(上述命令将自动打开网页)

![](/atts/trace.png)

- 采集heap
```
 go tool pprof http://localhost:6060/debug/pprof/heap
```

- 采集cpu
```
 go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

```
- 采集mutexes
```
go tool pprof http://localhost:6060/debug/pprof/mutex

```
### 详细Path

- /debug/pprof/allocs : 获取内存分配的抽样数据
- /debug/pprof/block ：获取导致同步阻塞的堆栈
- /debug/pprof/cmdline：获取当前运行程序的命令行路径
- /debug/pprof/goroutine：获取 goroutine 的执行堆栈
- /debug/pprof/heap：当前活动对象内存分配的抽样（也就是堆栈内存）
- /debug/pprof/mutex：获取互斥锁的持有者
- /debug/pprof/profile：获取 CPU 的抽样信息
- /debug/pprof/threadcreate：获取创建的线程信息（内核态线程）
- /debug/pprof/trace：获取当前程序执行的轨迹（获取 GMP 以 goroutine的调度信息

可供测试的完整简单代码如下

``` go 
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof"
)

func main() {

	go func() {
		log.Println(http.ListenAndServe("localhost:6060", nil))
	}()
	i := 0
	for {
		i += 1
		println(i)
	}
}

```

## bench 

```
go test -bench
```


## 参考
- https://go.dev/blog/pprof

