---
layout:     post
title:      "绕过 Windows 7 登录密码的小技巧"
subtitle:   "..."
date:       2017-05-17 12:00:00
author:     "Chris"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Windows
    - Security
---

一、Windows的登录界面

如下图所示，利用终端可以绕过登录界面，直接进入这堵墙背后的C盘，D盘，E盘，et al.

![](/img/2017-05-17-login-windowsOS-without-passwd/WechatIMG1.jpeg)

before that, 两个问题需要回答：

1. 怎样在登录界面上使用终端？

2. 终端出现了，然后呢？

二、Q1: 怎样在登录界面上使用终端

细心的童鞋会留意到，登录界面左下方有个“快速访问”的按钮，点开之后会出现各种小工具方便使用，比如“放大器”、“屏幕键盘”。

只要能将终端程序替代掉“放大器”，就能直接在这里调出cmd。

放大器对应的程序位于 C:\Windows\System32\Magnify.exe，所以，只要用 cmd.exe 替换掉 Magnify.exe 即可。

![](/img/2017-05-17-login-windowsOS-without-passwd/WechatIMG4.jpeg)

![](/img/2017-05-17-login-windowsOS-without-passwd/WechatIMG5.jpeg)

问题来了，桌面登不进去，怎么进行替换？

可以利用 U 盘做一个启动项，利用启动盘的系统使用功能管理整个磁盘，找到文件并替换。

过程中会遇到文件修改权限问题（sudo，你懂的）和 C 盘找不到的问题（利用 fdisk -l 查看挂在点，或直接 mount 挂在），总之最终结果就是：Magnify.exe 文件被 cmd.exe 文件成功替换。

![](/img/2017-05-17-login-windowsOS-without-passwd/WechatIMG2.jpeg)

三、Q2: 终端出现了，然后呢？

Windows 终端命令：

```sh
$net user USERNAME NEW_PASSSWORD
```
这是一个逻辑上异常神奇的存在，因为用这一命令可以修改密码，而且而且，修改密码时不需要输入原密码。。。

举个例子，这里楼长终端输入: net user USTCLI 123456

随后关闭终端，在登录界面密码栏输入：123456，密码认证通过，顺利进入。

以后再用电脑，直接上 123456 就可以了。

> 注：网络安全领域最困难的不是如何破门而入，而是破门而入后如何不被发现。本技巧是做不到这些的。


---

