---
title: '浅谈统计独立访客(UV或Unique Visitors)的策略'
categories: Computer Science
date: 2023-01-29 12:59:00
---
<!-- wp:paragraph -->

<p>统计独立访客一直以来都是互联网统计服务提供商们纠结的问题。这一问题曾经非常容易解决，但随着互联网的发展，精准统计独立访客反而变得越来越难了。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->

<h2>最粗暴，最不准确的方法</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->

<p>使用IP地址。是的。在最古早时代，所有连入互联网的设备都有一个独立的IPv4地址。那末，只需要把每一个用户的IP地址都存进一个数据库里，来一个存一个。如果发现是重复的IP就算作是同一个用户的多次访问。如果发现是不重复的那就是一个新的用户。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>所以我们说这个问题“曾经”很容易解决。然而由于云服务的兴起，IPv4的宝贵资源大多被云服务厂商抢去或被运营商保留，分配给用户的IP地址越来越少。对此，只能使用NAT(内网地址转换)来保障更多的用户能接入互联网络。这样一来使用IP地址作为用户的标识符变得不切实际了。更进一步，即使有一部分家庭网络出于某些原因确实有一个公共网络IP，它的下属有可能是链接了许多台设备的一个家庭局域网——而根据家庭的大小，由这种统计方式带来的误差最少是1,2个人，最大则可能有10多个人不等。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>最致命的是，现在的家庭宽带，即使有公网IP，也大部分是动态IP，这使得统计还要考虑IP的变化导致一个按照两个算。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>然而，IP地址数量虽然不能作为衡量独立访客的标准，却可以作为一个很好的网站性能指标。譬如一个网站声称可以接受2w+ IP的访问，就彰显了该网站服务器资源的雄厚。这是因为一般一个IP地址所带来的请求量是有限的，然而IP地址数量的上升则一定会带来请求数量的激增。所以用访问者IP地址的数量来帮助衡量网站的请求数量级。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>随着GeoIP数据库的完善，IP地址则带上了隐私的位置信息。如果采用明文记录，则可能带来信息泄漏的安全隐患。所以大部分的网站都应该拒绝明文记录访问者的IP地址。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->

<h2>歪门邪道且令人迷惑的方法</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->

<p>那么既然通过TCP这一传输层协议来实现对于用户数量的统计是有偏差的，用物理层的一些唯一标识符，比如MAC地址，如何呢？</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>然而这种想法虽然看起来很合理：MAC地址一般来讲都是唯一的，一般来讲都不会轻易改变——甚至可以用作用户的唯一标识符。可是实际上确实完全的馊主意。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>首先，同IP地址一样的缘由是，尽管很多人并不在意，MAC地址也是属于用户的隐私信息。如果明文存储这些信息，尽管利用它们的难度比利用IP地址干坏事的难度要大，也可能造成严重的后果。例如在商场连接了公共网络后，如果你的MAC地址泄漏，坏人就可以根据MAC地址反查到你的电脑品牌和外观——而这就意味着他们能够直接通过肉眼观察法找到你和你的设备。这还是最低技术含量的一种滥用MAC地址可能带来的后果。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>其次，很多路由设备会对原先客户端的MAC地址做一些不可预测的事。应该说，MAC地址是不是一定会存在于数据包的报头之中本身就是一个玄乎其玄的问题。如果用蜂窝移动数据，MAC地址可能会被去除或根本不会被发送。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>最后，就连用户自己也有权限随意更改他们的MAC地址。且这么做并不会影响到他们上网。而且这种做法还被鼓励为是一种保护隐私的策略。原因已经阐释过。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>当然这些原因都是“不推荐用MAC地址”的原因。还有一个最主要的因素是，在大部分情况下，是“不能用MAC地址”。目前在Web编程中获取用户MAC地址的方法实则相当有限。在服务端PHP提供的获取MAC地址的方法并不是广泛有效的（在Windows上工作在Linux上则永远也不会），而浏览器则将这种MAC数据作为用户的重点保护对象，除了像Internet Explorer这种电子古董允许通过JavaScript和ActiveX控件实现获取MAC地址以外，其他浏览器基本上不提供这类接口。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>不过还有一种例外。如果你的WEB服务有一个客户端，情况就大不相同了。可以在客户端上获取用户的MAC，然后和应用数据一并发送给服务端。但如果是用专用客户端来统计用户数据的方法，还有更多更好更全面的解决方案，也不一定需要使用这种具有隐私风险的解决方案了。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->

<h2>循规蹈矩的方法（被广泛采用）</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->

<p>所以我们还是回归实际，看看大部分的统计服务厂商是怎么做的。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>一般来讲，像Google Analytics这样的三方统计服务，直接给每一个访客生成一串专属于他们的UUID或者其他唯一标识符字符串，然后存储到他们浏览器的Cookie当中，这样如果他们反复访问站点，相同的Cookie被发送，经过数据库的比对，就可以确定是同一个访客。反之也就可以比较精确地统计独立访客了。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>然而这种统计方式的效果并不稳定，数据很容易虚高。典型的例子是用户清除了他们的浏览器缓存，用一些浏览器提供的所谓“一键清理”功能。这种情况下Cookies会被清除，于是他们就会被重新计算为一个新的独立访客。还有就是一个用户有多台设备或者一台设备上安装了多个浏览器，或者直接设置了每次关闭浏览器都自动清除所有Cookie。这样一来数据非常容易就变得很高。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>同时，数据还可能会骤然降低。在互联网上已经出现了很多宣传广告拦截器的内容。这类软件一旦安装在用户的计算机中，它们大抵不会被关闭或者暂停。这本来就对依靠广告维生的网站不太友好。然而对网站主更加糟糕的一点在于，大部分这样的广告拦截器拦截的不只是广告，还有第三方统计，譬如Google Analytics.它们已经维护了一份名单，对应了所有可能的追踪器url和cookie模式。一旦侦测到类似内容或行动就进行拦截，使得这些使用拦截器的用户在第三方统计面前就如同隐形人。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>针对cookie的用户端政策所带来的数据失真，最好的解决方法可能只是提示用户关闭他们的拦截器。然而如果网站上本没有广告，只是通过统计目的要求用户关闭拦截器是无理取闹的。且使用Google Analytics这类不自由的第三方统计系统本身就是不正当的，它们不是自由软件。更好的方案应该是就着问题解决问题，不要贪恋第三方的数据统计解决方案。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->

<h2>三方Cookie变单方Cookie</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->

<p>在Firefox，来自不同根域名的Cookie被判断为三方Cookie.譬如yep.example.com提供给www.helim.net的Cookie对Helim.net的用户来讲就是第三方cookie.但是yep.helim.net提供给blog.helim.net的cookie则被认为是本地的cookie.这种机制使得用户可以利用umami这样的简单搭建的统计平台，建设自己的自由开放的统计平台，且享受周全的数据展示。然而这种方式可能对SEO无利。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>如果用户使用Adguard这种专业的防追踪软件，来自不同子域名的Cookie也可能会被判断为三方Cookie.正如上一个例子中的后一者所讲，这回连yep.helim.net的cookie也被判断为第三方了。再糟糕一点，这些专业防止追踪的软件甚至会检测url中的文件名，直接拦截umami.js这样的必要的脚本文件，造成统计用户变得完全不可能。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>于是乎，一种完全基于单方的解决方案就有可能实施。对于一些视流量为生命的大站，这么做也完全合理。使用完全相同的域名，在CMS中实现一套或简单或复杂的统计系统。这样追踪拦截器将完全无法分析哪些网络请求是必要的，哪些则是用作统计用途的。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>只不过这种做法也可能会带来隐私隐患，而且可能耗费很多的精力，还带来用户的唾弃，由于这种不摆在明面上的数据收集。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->

<h2>总结</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->

<p>又免费又完全安全又完全准确的方案，目前是不存在的，除非你要求每一个访问网站的用户都登录一个账户并实名认证，然后根据账户操作来判断独立访客，否则100%精确的UV统计还不可能实现。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->

<p>目前的大流是使用Cookie或类似技术，然而IP地址统计依然可以作为数据估测和防止滥用的重要手段。</p>
<!-- /wp:paragraph -->
