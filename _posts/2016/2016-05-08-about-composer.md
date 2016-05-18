---
author: X
comments: true
date: 2016-05-07 14:00:00+00:00
layout: post
title:  使用composer管理依赖
categories: php

---

#### 什么是composer ?
**一句话来说，composer是一个依赖管理工具。类似于npm或者bundle.**


#### 为什么是composer?

##### 先看看没有composer之前的历史

composer之前，所有的依赖都是开发者手动维护的，也就是说，在开发项目的过程中，我们将会建立一个library之类的目录来存放引用的第三份库的完整copy。一旦，引用的第三份库进行升级，我们将不得不人工去升级。随着web越来越复杂，人工去维护依赖慢慢变成灾难，并且增加了发生错误的机会。一天晚上，一个家伙再也受不了这样的工作,composer诞生了。


##### 如果非要一个理由的话，额，如下：
- 自动管理复杂依赖，消除手动cp依赖带来的错误和巨大的时间精力消耗
- 主流框架均支持
- 这很酷不是么？😄

#### 快速的开始和结束

##### 安装
这里有详细的安装步骤 https://getcomposer.org/download/

##### root包和依赖包
root包和依赖包是一个相对概念，当你开发一个app，并且使用composer的时候，你当前的app就是一个root包。你所依赖的第三方的包就是依赖包。当你开发完毕的app被另一个app所依赖，那么你的app就变成另一个app的依赖包。

##### 说出你的依赖
所有的composer的依赖都在composer.json里面声明，你可以手动去创建这样一个json文件，也可以使用命令。

```   
    composer require "your dependency"
```
然后，使用`composer install`,就可以自动安装你声明的依赖。这个过程是根据composer.json的配置来寻找合适的源仓库进行下来。默认的package的仓库是在https://packagist.org/ 上。


##### 常见的命令


`composer init # 初始化`
`composer require "你的依赖" #声明依赖，将在composer.json里面增加对该依赖的声明`
`composer update  #更新依赖，将自动检测更新`，

##### 原理
composer.phar是composer的源代码的打包（类比jar）。composer是一个php程序，所以可以用 `php composer.phar 命令`的形式来执行。这是在全局没有安装composer的情况下的做法。
composer会将你声明的依赖下载到指定的目录，默认的，如在vendor下。然后自动生成autoload.在php支持namespace之后。如下的代码

`｀｀
    $f=new F;

`｀｀
注意，上面的代码，其实是一个相对的namespace（没有以\\开头的类都是相对namespace）在执行时候，将有一个搜索树来寻找F类的定义。这个顺序是：当前名称空间－>父namesapce->全局namesapce.

详细见这里：http://php.net/manual/zh/language.namespaces.rationale.php


```

<?php
namespace f;

class F {}

$f=new F;   #success

$f=new \f\F; #success

$f=new \F;   # Fatal error: Class 'F' not found

```
在上面定义的简单的类中，使用\f\F类似“绝对路径” 来访问类，最后一个\F,是因为在全局namesapce中并没有 F的类信息。
当调用一个不存在的类时候，php会具有完整namespace的调用信息传递给spl_autoload_register方法。该方法处理class和真实文件的映射关系。composer做了这部分工作。composer自动将它管理的第三份的库，注册进了autoload。

#### 总结
composer管理依赖。利用php的namespace的特性，实现了autoload.

#### 相关资源
https://getcomposer.org/  官网
http://www.php-fig.org/ [FIG小组]
https://packagist.org/ [package]
https://github.com/composer/installers#current-supported-types  自定义类型来安装到不同的位置
http://docs.phpcomposer.com/articles/handling-private-packages-with-satis.html  satis处理私有资源
http://docs.phpcomposer.com/articles/scripts.html 特定事件执行脚本
