---
author: admin
comments: true
date: 2015-08-25 09:05:25+00:00
layout: post
slug: zf2%e8%87%aa%e5%ae%9a%e4%b9%89%e7%9a%84controller
title: ZF2:自定义的Controller
wordpress_id: 257
categories:
- PHP
- Zend Framework 2
---

有个场景是这样的，在我的IndexController中，我想实现一个有参数的构造方法，这该怎么办呢？


传统的ZF2中，使用invokables来注册控制器。






    
    'controllers' => array(
        'invokables' => array(
            'Application\Controller\Index' => 'Application\Controller\IndexController'
        ),
    )




    
    如上所示是一个常见的控制器注册，但是如果在IndexController中添加携带参数的构造方法。



    
    class IndexController extends AbstractActionController
    {
    
        public function __construct($c){
        }
        public function indexAction()
        {
            return new ViewModel();
        }
    }




立即得到一个警告
Warning: Missing argument 1 for Application\Controller\IndexController::__construct(), called in X:\www\learn\vendor\zendframework\zend-servicemanager\src\AbstractPluginManager.php on line 207 and defined in X:\www\learn\module\Application\src\Application\Controller\IndexController.php on line 18



这时候就需要改变策略，使用工厂模式来创建控制器了。利用工厂模式可以更灵活的构造控制器，譬如初始化配置等。

    
    'controllers' => array(
        'invokables' => array(
        ),
        'factories'=>array(
            'Application\Controller\Index' => function(\Zend\Mvc\Controller\ControllerManager $cm){
                return new \Application\Controller\IndexController('Hello');
            }
    
        )
    ),







使用工厂模式的时候，一旦路由解析到对Application\Controller\Index的访问，将controllerManager对象作为参数传递给自定义的匿名回调中，controllerManager可以获取serverLocator等，在这里面可以进行自由的选择。



（尝试在construct中使用serverLocator的时候，抛出了错，于是引出这个话题，这个问题参见：http://stackoverflow.com/questions/18236468/zf2-getservicelocator-not-found）


