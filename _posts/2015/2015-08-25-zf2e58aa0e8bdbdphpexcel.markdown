---
author: admin
comments: true
date: 2015-08-25 02:00:53+00:00
layout: post
slug: zf2%e5%8a%a0%e8%bd%bdphpexcel
title: ZF2:加载PHPExcel
wordpress_id: 253
categories:
- Zend Framework 2
---

今天在项目中使用了PHPExcel,遇到一点问题，记录一下。

首先是如果将PHPExcel注册到ZF2里面。我在使用class_map将PHPExcel注册到名称空间里面。但是会遇到奇怪的问题，譬如当我使用PHPExcel_IOFactory的时候居然提示我找不到该类.

原来，我必须通过new PHPExcel来触发PHPExcel的spl_autoload_register并注册PHPExcel_IOFactory，这是个非常奇怪的现象。但知道PHP对namespace的处理之后，不难理解,我在class map中注册了，这并没有将PHPExcel的spl_autoload_register注册,仅仅是我同过new的时候才触发了PHPExcel的自动注册。

一种解决方案是，在自定义的autoload中将PHPExcel的自动加载也注册进去,譬如这里给出的解决方案 :

[http://stackoverflow.com/questions/9072687/phpexcel-fatal-error-class-phpexcel-shared-zipstreamwrapper](http://stackoverflow.com/questions/9072687/phpexcel-fatal-error-class-phpexcel-shared-zipstreamwrapper)。

最简单的办法就是使用composer啦，这是依赖管理神器。

1 编辑composer.json ,添加以下行：

    
    "repositories": [{
    "type": "package",
    "package": {
    "name": "PHPOffice/PHPExcel",
    "version": "1.8.0",
    "source": {
    "url": "https://github.com/PHPOffice/PHPExcel.git",
    "type": "git",
    "reference": "1.8.0"
    },
    "autoload": {
    "psr-0": {
    "PHPExcel": "Classes/"
    }
            }
        }
    }],
    ,
    "require": {
        //......
    "PHPOffice/PHPExcel": "1.8.*"
    }


2 执行（管理员权限，确保可写） php composer.phar update  稍等一会儿就OK啦（可能需要selfupdate下）



在不方便使用composer的情况下，可以通过其他的方式，核心是，你需要想办法调用PHPExcel_AutoLoader的register方法，该方法将进行spl自动加载注册。如你可以在index.php中增加require('path/to/PHPExcel.php');但这并不是一种优雅的做法。
