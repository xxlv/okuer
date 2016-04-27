---
layout: default
title: About
---

<link rel="stylesheet" type="text/css" href="{{ site.url }}/assets/static/css/index.css">
<script src="{{ site.url }}/assets/static/js/zepto.min.js"></script>

<div id="container">
<div id="guidePanel"></div>
<div id="gamepanel">
    <div class="score-wrap">
        <div class="heart"></div>
        <span id="score">0</span>
    </div>
    <canvas id="stage" width="320" height="568"></canvas>
</div>
<div id="gameoverPanel"></div>
<div id="resultPanel">
    <div class="weixin-share"></div>
    <a href="javascript:void(0)" class="replay"></a>
    <div id="fenghao"></div>
    <div id="scorecontent">
        您在<span id="stime" class="lighttext">2378</span>秒内抢到了<span id="sscore" class="lighttext">21341</span>个月饼<br>超过了<span id="suser" class="lighttext">31%</span>的用户！
    </div>
    <div class="textc">
        <span class="btn1 share">请小伙伴吃月饼</span>
    </div>
</div>
</div>

<script src="{{ site.url }}/assets/static/js/index.js"></script>
