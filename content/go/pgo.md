---
title: "go1.20 Pgo优化"
date: 2023-03-04T15:32:55+08:00
---

# 介绍

> Profile Guided Optimization (PGO) 是一种编译器优化技术，通过基于应用程序的运行时间信息优化生成的机器代码。具体来说，PGO 通过使用编译器生成的剖面数据文件（即有关程序在运行时执行情况的信息），来指导机器代码的优化过程。这些信息可以帮助编译器更好地理解应用程序的性能瓶颈，并生成更高效的代码。

PGO 通常分为三个步骤：
- 使用编译器创建一个基准版本的可执行文件；
- 在训练阶段中运行此基准文件，并收集各种性能统计数据，如函数调用频率、循环迭代次数等；
- 使用这些统计数据重新编译程序，并应用优化策略，以生成经过改进的代码。

相比于传统的静态编译优化方式，PGO 的优势在于，它可以根据实际运行情况进行优化，因此可以更准确地优化重要的代码路径，而不是仅仅优化一些假定的热点代码。因此，PGO 可以提供更好的性能和更低的代码大小，尤其对于大型项目或者需要处理大量数据的应用程序，效果更加明显。


# In Go 
> Go 1.20版本于2023年2月份正式发布，在这个版本里引入了PGO性能优化机制。

## 步骤

1.  引入 pprof 

``` go 
    // 项目中引入
    _ "net/http/pprof"
```

2. 采集
`curl -o cpu.pprof "http://localhost:8080/debug/pprof/profile?seconds=30"`

3. 修改为 `default.pgo` 执行 `mv cpu.pprof default.pgo`

4. go build -pgo=auto -o pro.withPgo

> auto 模式将自动的使用 `default.pgo` 来进行优化 默认-pgo 为 `off` 

###  压力测试

####  生成 统计结果
-  go test load -bench=. -count=20  > a.txt 
#### 比较统计
-  benchstat  a.txt  b.txt

> 安装 `go install golang.org/x/perf/cmd/benchstat@latest`

