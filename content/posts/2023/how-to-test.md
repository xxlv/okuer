+++ 
date = 2023-03-01T21:40:08+08:00
title = "使用Locust做压力测试"
description = "Python  包"
slug = ""
authors = []
tags = []
categories = ["Python","Test"]
externalLink = ""
series = []
+++


## Locust 压力测试
Locust 是一个开源的 Python 压力测试工具，它可以模拟大量用户并发访问 Web 应用程序，测试其性能和稳定性。以下是 Locust 的使用步骤：

1. 下载和安装
使用 pip 安装：在命令行中运行 pip install locust 即可安装 Locust。
从源代码安装：可以从 Github 上下载最新的源代码（https://github.com/locustio/locust），然后运行 python setup.py install 进行安装。
2. 编写脚本
Locust 脚本是用 Python 编写的，它定义了压力测试的场景、用户行为和请求流程。下面是一个简单的 Locust 脚本示例：

```python
from locust import HttpUser, task, between

class MyUser(HttpUser):
    wait_time = between(1, 3)

    @task
    def my_task(self):
        self.client.get("/")
```

在这个示例中，定义了一个名为 MyUser 的 Locust 用户类，它继承自 HttpUser 类，并定义了一个名为 my_task 的任务方法。任务方法使用 self.client 对象来发送 HTTP 请求。wait_time 属性指定了任务之间的等待时间。

#### 运行测试
命令行方式：在命令行中进入脚本所在目录，然后运行 `locust -f script.py` 命令启动 Locust。
Web 界面方式：在命令行中运行 `locust -f script.py --web-host=0.0.0.0` 启动 Locust Web 界面，在浏览器中打开 `http://localhost:8089` 即可访问 

## Locust Web 界面
Locust Web 界面提供了一些方便的功能，如：

单击 “Start Swarming” 按钮以开始测试。
1. 在 “Statistics” 标签页中查看有关测试结果的详细信息。
2. 在 “Charts” 标签页中查看有关测试结果的图表。
3. 在 “Failures” 标签页中查看请求失败的详细信息。
4. 在 “Settings” 标签页中配置测试参数，如并发用户数和持续时间。


