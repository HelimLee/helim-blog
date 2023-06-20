---
title: '加上这个Meta属性，保护隐私！'
date: 2023-06-04 21:16:50
cover: /images/no-refer.png
tags:
- 技术
---

日常生产生活中，很多人看HTTP标头的时候都会发现一个这样的字段：

```
Referer: https://example.com/example
```

这个`Referer`字段是什么意思呢？其实这是一个HTTP协议设计的“历史遗留问题”。因为当时开发的程序员可能是熬夜了，眼睛昏花，敲错了字，把原来的`Referrer`，也就是“来源”，敲成了`Referer`。后来有关方面曾经尝试过修复，并推出了新的标头，毕竟HTTP协议这种东西作为现代互联网基石最好严谨些。但无奈这个标头已经广泛推送给了大多数客户端和服务端，所以修改进程遇到很大的阻力，最后宣布失败，新标头被弃用并移除。

## Refer(r)er 的危害

懂英语的人可能看懂了本文封面的那行代码：即禁用浏览器向HTML引用的资源服务器发送Referrer. 这种情况下就可以实现隐私保护。

现在假设有一个A网站，没有设置这行代码，而A网站中嵌入来自B网站的某个css文件。浏览器照常工作，把标头发送给B网站的服务器。但出乎所有人意料的是，B网站是一个对隐私极其不友好的网站，它把所有请求Referrer全部记录下来。因为B网站上的所有页面都引用了这个CSS。所以可以说，B网站现在有方法可以统计A网站的访客数量，以及专门针对A网站的访客进行进一步的隐私冒犯，这是非常危险的。（在这种情况下，对于B网站而言，用户们从“互联网用户”分为“A网站的用户”和“其他网站的用户”，而后者还可以进一步划分成D网站、E网站的用户等等，这种行径往往是进一步隐私冒犯的前兆）

而且，不只是外联引入中发送Referrer这种隐蔽形式，连**超链接**也会发送，用户点击超链接的时候可能没有想到，因为所有人都觉得这种手段也太明目张胆了。但是请看：现在A网站上有一个链接指向了B网站的某个页面，当用户点击A网站的链接跳转到B网站时，B网站竟然能获知这个用户是从哪里来的。其实这就是**网络追踪**，而且还是最低劣的手段，且一般不会被常见的追踪拦截器拦截，就取得了用户具体的网络浏览路径。无疑是大大降低了跟踪成本。

## 采取措施

Referrer用处很大，最出名的当属防盗链。所以不可能指望废除Referrer字段。但在现代浏览器中，（当然不包括IE）都可以通过一句话的HTML来实现禁止浏览器向外部资源发送该字段。根据[Can I use](https://caniuse.com/referrer-policy)的报告，看起来就是IE也支持设定为`origin`，虽然这个模式对隐私保护的作用不大。

根据[MDN的文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)，通过`Referrer Policy`实现了对是否发送`Referrer`字段的控制。具体需要通过一句HTML代码，在`<head>`中实现

```html
<meta name="referrer" content="${The Policy}" />
```

目前一共支持包括不发送任何`Referer`字段的`no-referrer`和只发送主机名的`origin`在内的共八种政策。但前者是对隐私保护最有利的，因为其他类型都会或多或少，或具体或模糊地对外发送Referer.

但在采取措施时还需格外注意，将`Referer`字段置空意味着无法再通过设置白名单为仅包含自己的域名来防盗链，否则自己的网站上也无法显示自己的资源。所以还需谨慎设置。