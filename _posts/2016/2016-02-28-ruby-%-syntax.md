---
author: X
comments: true
date: 2016-02-27 03:49:00+00:00
layout: post
title: Ruby的%Q,%q,%w,%s,%x
categories: ruby
---


###### %Q(%q)
优雅的转义方式：当你使用字符串的时候，你可能经常需要这样的代码：

{% highlight ruby %}

    >> %Q("""hello,world!""")
    => """hello,world"""
{% endhighlight %}

你不需要为双引号编写额外的转义符号。％q将对单引号起作用。
你还可以编写更加优雅的代码
{% highlight ruby %}

    s=%Q(""hello ,world "")  #=> ""hello ,world ""
    s=%Q+""hello ,world ""+  #=> ""hello ,world ""
    s=%!""hello ,world ""!   #=>  ""hello ,world ""
    s=%Q!'''hello ,world'''! #=> '''hello ,world'''
    s=%!'''hello ,world'''! #=> '''hello ,world'''

{% endhighlight %}


##### 其他的几个%x
- %w(%W),表示数组
{% highlight ruby %}
    s=%w(a b c )   #=>  [a,b,c]
    s=%w(\ ab c )   #=> [ ab,c] 空格被转义
{% endhighlight %}

- %x,表示shell
- %s,表示symbol , %s(#{var}) 并不会转换var哦！
- %i,生成一个symbol数组
