---
author: X
comments: true
date: 2016-04-05 14:00:00+00:00
layout: post
title:  建立一个sftp服务
categories: computer

---

hero告诉你，他需要在ubuntu上搭建一个sftp，将/data目录共享给hero这个账号。这个账号的密码是123456，你接受了入职以来的第一个巨大任务。并且衷心告诫hero，这个密码并不安全。他并没有接受你的建议（你心里自然愤愤）。

好了，你懒得跟他扯，因为你知道有一天他迟早来找你。现在你获得来具有root权限的账号，并登陆了ubuntu的terminal。是时候开始这个任务了。

嗯，经过大脑对信息的快速判断，你立即分解出更小的任务：

1.  创建一个名叫hero的用户，并且设置密码为123456

2.  创建一个sftp server，将hero加入策略组（你完全不知道该怎么办的艰巨任务）

第一个任务似乎很容易就能做完，你只用了下面的几条命令：

{% highlight ruby %}
sudo useradd hero  #创建一个名为hero的用户
sudo passwd hero   #给hero添加密码
{% endhighlight %}


ok，做完上面两步之后，50％的任务已经完成，但事情渐渐变得棘手，你完全不知道sftp是一个什么技术，你只对ftp了解。你的猜测是：

（跟大多数服务一样）sftp跟ftp类似，它运行在服务器上，监听某个端口。

基于这样的猜测，你在google输入：sftp。在2分钟内查询到了全部信息。基本上跟你猜测一样。

现在，你已经确认你安装了sftp服务，开始配置它，支持你的用户。你找到配置文件位于：

{% highlight ruby %}
/etc/ssh/sshd_config
{% endhighlight %}


在行末新增：
{% highlight ruby %}
Match Group sftponly        #匹配sftponly组
 ChrootDirectory  /data     #目录
 ForceCommand internal-sftp #强制执行这里提供的命令
 AllowTcpForwarding no      #禁止tcp转发
 {% endhighlight %}



这个配置项将匹配到sftponly组，将目录映射到/data下



等等，你似乎忘记了一件重要的事情，你需要把hero加入到sftponly这个组内。使用命令:

{% highlight ruby %}
sudo usermod  hero -a -G sftponly
{% endhighlight %}

执行完毕，可以在

{% highlight ruby %}
cat /etc/group
{% endhighlight %}

来查看执行的结果。

ok,现在用
{% highlight ruby %}
sftp hero@yourhost
{% endhighlight %}

可以发现，hero已经成功的登陆进来了。



## 回顾

截至目前，我们完成了这个任务的基本需求。但是，等等，这里有一个非常重要的部分，那就是data目录的权限问题。

1. 一定要确定data目录的拥有着是root(why?)

2. 一定要保证没有群组的w权限(why?）



下面是一种可行的权限设置：

{% highlight ruby %}
sudo chown root:sftponly /data
sudo chmod 755 /data #r=4 w=2 x=1
{% endhighlight %}


## 总结

**注意权限**
