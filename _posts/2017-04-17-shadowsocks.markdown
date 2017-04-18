---
layout:     post
title:      "利用 Shadowsocks 观看完整版「人民的名义」"
subtitle:   "城里的人想出去，城外的人想进来，that's it."
date:       2017-04-17 12:00:00
author:     "Chris"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Google
    - Shadowsocks
---

> 最近身边有朋友剧荒了，本着好剧大家看的原则写下此文，以介绍如何登陆 Youtube 继续追剧事业。闲话少说，直接上图：

![](/img/2017-04-17-shadowsocks/shadowsocks-youku.png)

## 基本原理

要访问需要的数据，其实原理很简单：“长城”使用的是黑名单制。设想数据由 A 传到 B 被阻止，那是因为 A 和 B 之间有道墙，墙上挂着告示（名单），上说 A 被通缉，不得入内。可是，如果把 A 的数据传给 C，再由 C 传给 B 不就可以了吗？

这里有一点需要注意，那就是 C 和 A 之间不能再有“告示”，所以，C 的地理位置很重要，你懂的。

![](/img/2017-04-17-shadowsocks/shadowsocks-earth.png)

楼长的做法是，首先在洛杉矶搭建一台服务机，然后在大洋两端分别完成 Shadowsocks client 和 Shadowsocks server。数据流将通过一台服务器的中转进来。一切完成之后，就可以继续追剧了。

除了《人民的名义》，我们也可以看到更多的新闻资料和技术材料，如 Google Scholar 等。

网上的所有言论都是由人发布的，而每个人的深层动机和政治考量各不相同，相信广大同胞凭借自己的智慧足以分辨，当然这一点和技术无关。

![](/img/2017-04-17-shadowsocks/shadowsocks-cnn.png)

## Shadowsocks Server

一.&nbsp;获取硬件资源

首选当然是 Google cloud platform，谷歌为了广大老百姓也是操碎了心。遗憾的是，要访问 Google 服务器就要先 FQ，而如果能 FQ 还要 Google 干嘛？这是个悖论。

所以，这里我在其他榜上无名的服务商那里租来了一台小型服务器，用于满足个人需求（服务器的 CPU 和 Memory 均没有要求，但网络带宽要靠谱）。这里我选择使用的是 [RFCHOST][1]。本文并不是什么广告贴，大家用其他任何的服务商都是没有问题的。

![](/img/2017-04-17-shadowsocks/shadowsocks-f.png)

不过还是在啰嗦一句，租用服务器时建议将如下几点列入考虑范畴：

- 价格，当然要便宜些，服务器一般有年租和月租之分，不知水深的最好短期租用；

- 选择口碑好的服务商

- 另外（划重点），<b>服务器一定要在境外、香港或天朝的台湾省</b>。

网上有很多为了新注册会员免费试用数月的优惠，所以即便是服务器也是可以淘到免费的。给大家个市场价，月租 5$ 左右，就足够拿到一台满足自己需求的服务器。

> 以下开始技术部分！

二.&nbsp;搭建服务器架构

这里默认大家已经有一台服务器了，那么怎样在上面搭建 Shadowsocks 服务呢？这里楼长以 Ubuntu 16.04 为例介绍。

首先当然是进入系统，更新一下 apt-get 软件包，这一步是所有系统安装成功后都要做的。

```sh
$sudo apt-get update
```

随后，通过 apt-get 安装 python-pip

```sh
$sudo apt-get install python-pip
```

完成之后使用 pip 安装 shadowsocks 服务

```sh
$sudo pip install shadowsocks
```
成功安装后，我们需要创建一个shadowsocks server 的配置文件，可以直接建在当前用户目录下

```sh
$sudo vim ss-conf.json
```

然后输入

```c
{
    //使用当前服务器，填0.0.0.0
    "server":"0.0.0.0",
    //对外的服务端口,随意
    "server_port":8888,
    //本地IP和端口
    "local_address":"127.0.0.1",
    "local_port":1080,
    //shadowsocks密码，自己设置
    "password":"youpassword",    
    "timeout":600,
    //加密方式
    "method":"aes-256-cfb"
}
```

保存。

最后，利用这个配置文件启动 shadowsocks 服务。

```sh
$sudo ssserver -c ss-conf.json -d start
```

至此，服务器的配置也就完成了，简单吧！

当然，我使用的是裸服务器，对于使用 google cloud 服务器的童鞋请注意，由于 google cloud 自带防火墙，所以我们需要在防火墙规则里加入端口白名单，否则外部是无法访问这个端口的服务的。

修改防火墙设置的方法官方给出了详细说明，如下图，也可以请教[度娘][2]，这里就不再赘述。

![](/img/2017-04-17-shadowsocks/shadowsocks-google-platform.png)

## Shadowsocks client

如此一来，大洋另一端的服务器也就完成了（Shadowsocks server），下面来看看本地的客户端（Shadowsocks client）。

由于客户端的多样性，这里分别介绍：

<b>一.&nbsp;Windows</b>

首先介绍 Windows，需要注意，在 Windows 除了安装客户端，还要对浏览器进行额外配置。这是 Windows 比其它操作系统要复杂的地方。

通常情况，我们希望浏览器在访问的国外网站的时候使用shadowsocks，而访问国内网站的时候使用本地网络，比如你在优酷看视频，并不需要走一圈代理，这样速度反而会更慢。那么安利一个 chrome 下 shadowsocks 的黄金搭档，SwitchyOmega。

![](/img/2017-04-17-shadowsocks/shadowsocks-switchyomega.png)

这个可以在 chrome 应用商店里下载安装。（配置的复杂程度和当初配 goagent 相当，求阴影面积）

---

深吸一口气，下面开始介绍本文最晦涩的一部分：<b>Chrome</b> + <b>SwitchyOmega</b> + <b>Shadowsocks</b> 的黄金组合！

1.&nbsp;电脑端下载 shadowscoks([最新版下载链接][4])，打开后开始配置，第一步选择 “编辑服务器”

2.&nbsp;在这里配置上你获得的 shadowsocks 的IP 端口 和密码，代理端口可以只有选择的，不过一般都选择 1080。

具体配置的内容，和接下来介绍的 macbook 上的配置一样一样的，除了界面丑陋点。

3.&nbsp;点击下 “<b>从GFWLIST更新本地PAC</b>“ 更新完PAC后选择 <b>使用本地PAC</b>。

![](/img/2017-04-17-shadowsocks/shadowsocks-1.jpeg)

4.&nbsp;系统代理模式 选择 <b>PAC模式</b>

![](/img/2017-04-17-shadowsocks/shadowsocks-2.jpeg)

5.&nbsp;正常来说运行了ShadowsocksR后就可以直接访问Chrome应用商店安装了（[在 Github 上下载最新版安装包][5]）。

chrome 安装好 [switchyomega][6] 后选择 <b>新建情景模式</b>，<b>情景模式名称</b> 可以随便选择 模式类型选择第一个 <b>代理服务器</b>

![](/img/2017-04-17-shadowsocks/shadowsocks-new.png)

6.&nbsp;代理服务器按下表配置就okay了，到此也就可以正常使用了PAC模式了。但是为了更进一步更方便的使用我们还可以增加一步自动切换。

![](/img/2017-04-17-shadowsocks/shadowsocks-3.jpeg)

7.&nbsp;选择右边的 <b>自动切换/auto switch</b> 最下面的 <b>规则列表设置</b> 按下表设置就好了 规则网址也是我图上给出的这个。

![](/img/2017-04-17-shadowsocks/shadowsocks-4.jpeg)

这里，AutoProxy 规则列表地址有所变动，新的地址是: [download][11]

8.&nbsp;然后最关键的一步，在浏览器右上角点击插件按钮，然后选择自动切换规则模式，这样只要在规则列表里面的网站都会翻墻访问。

*本部分参考：*

*[Chrome+SwitchyOmega+Shadowsocks 完整篇][7]*

*[GFWList · FelisCatus/SwitchyOmega Wiki · GitHub][8]*

---

<b>二.&nbsp;Macbook</b>

Mac 和 Windows 一样，都有 Shadowsocks 客户端，可以在这里[下载][3]。配置的话就像这样输入，然后启动

![](/img/2017-04-17-shadowsocks/shadowsocks-mac.png)

虽然很多教程声明，安装完成后不需要重启计算机即可使用服务，但从楼长的实际操作来看，还是要重启电脑的。

<b>三.&nbsp;Android</b>

Android 的安装是最简单的，IP 等配置和前面介绍的完全相同。

这里只给出下载地址：[［Download］][9]

对了，Shadowsocks 还有个中文名字，叫“影梭”，换了马甲还是要认识的。

<b>四.&nbsp;Linux</b>

由于撰写本文时楼长的 linux 电脑没开，也懒得开了，这里就不截图了。

安装命令如下：

```sh
$sudo add-apt-repository ppa:hzwhuang/ss-qt5
$sudo apt-get update
$sudo apt-get install shadowsocks-qt5
```

安装之后对于浏览器的配置与 Windows 类似。

## 关于本网站的评论

在楼长的第一篇博客中介绍过本博客是如何搭建起来的，但当时对于博客重要的一环：<b>评论系统</b> 按下不表。

这其实也是 FQ 的问题，其实楼长在搭建本系统时是嵌入了评论系统的，由于国内最大的评论系统 <b>多说</b> 在即将到来的六月一号就要关闭，我选择使用了美帝的 DISQUS，有图有真相：

![](/img/2017-04-17-shadowsocks/shadowsocks-disqus.png)

所以，如果作为吃瓜群众没能看到评论，说明还没 FQ 成功，后续楼长会把评论系统部分重新设计...

## 背景

[如何看待2015年8月20日 shadowsocks 作者停止维护的事件? - Shadowsocks - 知乎][10]


以上。

---

[1]: https://my.rfchost.com/
[2]: https://www.baidu.com/s?wd=google%20cloud%20platform%20%E9%98%B2%E7%81%AB%E5%A2%99%E4%BF%AE%E6%94%B9&rsv_spt=1&rsv_iqid=0xeb483baa0000adc7&issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_sug3=46&rsv_sug1=3&rsv_sug7=100&rsv_t=3c7dR0oOboXLJECGPejbICEbc781wBiWLqGjKBK2v46p16cQs%2FFILl85DLcYUekuhTtj&rsv_sug2=0&inputT=9213&rsv_sug4=9214
[3]: https://pan.baidu.com/s/1jIkeUPo
[4]: https://github.com/shadowsocksr/shadowsocksr-csharp/releases
[5]: https://github.com/FelisCatus/SwitchyOmega/releases
[6]: https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?utm_source=chrome-app-launcher-info-dialog
[7]: http://blog.sina.com.cn/s/blog_76ed34130102vl8r.html
[8]: https://github.com/FelisCatus/SwitchyOmega/wiki/GFWList
[9]: https://github.com/shadowsocks/shadowsocks-android/releases
[10]: https://www.evernote.com/shard/s9/sh/4b3fda4c-866b-4025-987f-8328f28d6d1b/ad5beb3531adb7b7f33b30903fd6b0ff
[11]: https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
