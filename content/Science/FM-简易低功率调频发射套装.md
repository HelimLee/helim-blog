---
title: 'FM:简易低功率调频发射套装'
categories: Radio Technology
date: 2022-01-13 20:23:00
tags:
- 低功耗无线电
- 技术
---
<!-- wp:html -->
<div class="wp-block-argon-admonition admonition shadow-sm" style="border-left-color:#f75676"><div class="admonition-body">这是低功率发射装置，本装置设置功率为150mW,请读者在效仿时注意发射功率根据国家要求，在不准入频段无证无呼号发射无线电是不得超过1W的（1W=1000mW），请自重</div></div>
<!-- /wp:html -->

<!-- wp:html -->
<div class="wp-block-argon-alert alert" style="background-color:#f75676"><span class="alert-inner--icon"><i class="fa fa-info-circle"></i></span><span class="alert-inner--text">在家庭或其他形式的非专业室内操作大于运行总电压12V的电路将会是有危险的。操作大于36V的电路则有可能致命。继续阅读本文即代表您承诺自行负担所有风险。</span></div>
<!-- /wp:html -->

<!-- wp:more -->
<!--more-->
<!-- /wp:more -->

<!-- wp:paragraph {"fontSize":"medium"} -->
<p class="has-medium-font-size">首先一切的一切是一个FM发射模块。这里我不推荐，一是不做广告，二是我自己买的这个非常不尽人意，功能没有其宣传的那么有用。如果让我再买一次的话我会选择全串口控制的版本（<del>而且更加能够装B</del>而且还更加便宜）。先给大家看一下我的模块外观</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"center","fontSize":"medium"} -->
<p class="has-text-align-center has-medium-font-size"><br><img style="width: 500px;" src="https://helim-edge-endpoint.azureedge.net/helimstorage/9/IMG_20220113_185338.jpg" alt="FM图片"><br>（从左到右的几个按键依次是VOL- , VOL+ , FRE- , FRE+ , PAUSE 可以控制音量和发射频率，以及暂时性关闭发射）<br>然后是一个收音机，为了测试是否成功发射信号。收音机有三种选择，一种是搜索台的收音机，一种是手动旋钮或者按键输入的，还有一种是上面两种的综合体。我选的就是最后一种<br><img src="https://helim-edge-endpoint.azureedge.net/helimstorage/9/IMG_20220113_183252.jpg" alt="收音机" style="width: 500px;"><br>虽然这个收音机看起来像个老人播放器（<del>实际上就是老人播放器</del>），但是其功能毫不逊色。不过某些人士可能拥有手台/车台，这是极好的，不过拥有这种东西的人应该都是有证有呼号的专业无线电玩家，我就不插嘴了。<br>为了让你的信号通过有限的功率传递更远，一个优秀的“天线”是必要的。为什么“天线”要打双引号呢？不是因为可能不够优秀，而是因为可能根本长得就不像天线！FM为什么能够拥有数量众多的电台，很大程度就是因为其天线的极易制作。天线是这么做的：</p>
<!-- /wp:paragraph -->

<!-- wp:html -->
<div class="wp-block-argon-alert alert" style="background-color:#7889e8"><span class="alert-inner--icon"><i class="fa fa-info-circle"></i></span><span class="alert-inner--text">如果您对您家庭电路系统足够自信，且愿意自行承担可能带来的后果，您可以使用家庭电路中的地线作为天线。</span></div>
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>如果你认为只要实用就行的话，可以找来一根电线，这根电线越长越好，如果你不怕死的话直接连楼房里的地线更好（危）。接着把这电线一头削掉，裸露的部分用点焊（我是直接别上去）焊在FM模块的ANT接口上（别瞎接，否则就变成又不好看还不实用的破线了），接着把这根长电线丢到窗外去，你的信号就可以传到很远（实测5楼传到方圆楼下30多米没什么毛病，<del>150mW你还想要什么</del>）的地方了。我上次捣鼓单片机买了一堆杜邦线，我就把这些杜邦线连在一起也成了长电线，效果极佳。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>如果你认为好看一点，科技感一点能给你带来更好的成就感的话，看这个WikiHow的教程<br>如果你认为你很有钱，直接买一个八木天线吧！住别墅的可以直接放楼顶，还附带避雷针功能（bushi）。</p>
<!-- /wp:paragraph -->