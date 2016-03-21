---
author: X
comments: true
date: 2016-03-20 04:25:00+00:00
layout: post
title:  从0开始制作一个gem
categories: ruby

---

gem是一种软件组织形式，对ruby源程序提供一个包机制。rubygem是对这包进行管理的工具，也就是在命令行中常用的gem。
如何创建一个gem呢？
首先，生成必要的目录结构。这个结构可以手动创建，也可以通过bundle。
使用bundle的一个例子是：


{% highlight ruby %}
bundle gem my-gems
{% endhighlight %}

这条命令将生成以下的目录结构：

{% highlight ruby %}
my-gems/Gemfile
my-gems/.gitignore
my-gems/lib/my/gems.rb
my-gems/lib/my/gems/version.rb
my-gems/my-gems.gemspec
my-gems/Rakefile
my-gems/README.md
my-gems/bin/console
my-gems/bin/setup
{% endhighlight %}

这是一个标准的gem目录结构，全部的配置都在my-gems.gemspec里面完成，可以看到my-gems.gemspec里面的配置使用了ruby语言描述，因为ruby太像一个伪代码。

##### 一些基本的配置项整理如下：

- spec.files #指定的文件将被包含，注意这里如果用`git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
那么，你创建的文件要加入到gem的话，需要让git先跟踪。

- spec.executables #可执行文件，将自动添加到$PATH包含的那个目录(如/usr/local/bin)


抛开配置的细节，下面来创建
{% highlight ruby %}
gem  build my-gems.gemspec

#Successfully built RubyGem
#Name: my-gems
#Version: 0.1.0
#File: my-gems-0.1.0.gem
{% endhighlight %}

将生成my-gems-0.1.0.gem这个文件，ok这个文件是可以被装载的。具体的命令为
{% highlight ruby %}
 gem install my-gems-0.1.0.gem
{% endhighlight %}

当执行上面的这条命令，便会在本机上装载gem。如果你想要让更多人看到，可以发布到rubygems上。
gem push my-gems-0.1.0.gem
噢对了，你需要先注册一个账号.


##### 链接
http://guides.rubygems.org/make-your-own-gem/     
