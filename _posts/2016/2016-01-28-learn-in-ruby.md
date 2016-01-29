---
author: X
comments: true
date: 2016-01-25 03:49:00+00:00
layout: post
title: Ruby一些基础知识笔记
categories:
- Ruby
---
### Ruby编程学


#### Ruby的类

-  class是一个表达式，class申明类，将创建一个常量，所以类名的首字母是大写的
-  attr_accessor,attr,attr_reader　访问存取器
-  @x,@y 实例变量，initialize()构造器
-  可定义操作符方法
-  each方法

#### 内置方法

- initialize 构造器
- each 可被迭代
- <=>方法　 Mixin Comparable
- eql? 和　==

####　可变类
- `Struct.new("Point",:x,:y)`
- `Point=Struct.new(:x,:y)`


#### 类的一些特性
1. public,private,protected 实际上是Module的一个方法,class是Module的子类
无法定义一个在类的外部无法访问的常量，也无法将一个实例变量在外部访问，除非使用了访问器(attr_accessor等)
2. 对于类方法，要想在外部不可访问，譬如在使用单例模式，隐藏MyClass.new的调用，此时可以使用
private_class_method :new
3. 两种方式可以访问私有的方法
*  obj.instance_eval { my_private_method }
*  obj.send :my_private_method


4. 覆盖私有方法,对类成员，子类和父类同名的类成员会修改父类的类成员，这个影响对所有继承该类的子类都起作用
5. new方法发生了两件事：创建一个对象，并且初始化
6. class << self 定义的class method 无法访问mixin的变量
7. public ,private,protected,module_function（将实例方法变为私有方法） 都是Module的私有方法

#### 创建对象的方式
+  new (allocate and initialize) ,factory
+  clone , dup and initialize_copy
+  Marshal.load (Marshal.dump)
+  Singleton
+  一个对象的单例方法是关联它的匿名Eigenclass的实例方法



#### 加载方式

- load >> load直接载入，第二个参数不是nil和false ，会wrap到一个匿名的模块中，保证上下文的安全（防止名称空间的紊乱）
- include　
- require >> 类似php的require，会做检查，争取最少的加载
- autoload >> 惰性加载,autoload :MYMODULE , "hello" ,在第一次使用MYMODULE的时候才加载



#### Ruby常见的几个术语

- [RVM](https://rvm.io/rvm/install)

- [Ruby on Rails](http://rubyonrails.org/)

- [RubyGems](https://rubygems.org/)

- [Gem](https://rubygems.org/)

- [Rake](https://github.com/ruby/rake)  make-like build utility for Ruby

- [GemFile](https://rubygems.org/)

- [RakeFile](https://github.com/ruby/rake)

- [Bundle](http://bundler.io/)

- ri

### Ruby 工具集


#### Gem做了什么？
在终端输入：
`sudo gem install jekyll` 发生了什么？


1. gem命令将启动一个ruby程序，这个程序的源代码被定义在/usr/bin/gem中．可以在linux中通过 `whereis gem`来发现它的位置．其实本质是一个ruby程序．

2. Rubygems 将在sources中查询jekyll并下载指定的gem,sources可以在　`gem sources -l`中查询，默认的源比较慢，切换到淘宝源，可以用 `gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/`
