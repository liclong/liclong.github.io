---
layout:     post
title:      "如何实现分布式文件系统HDFS到本地Linux文件系统的映射"
subtitle:   ""
date:       2017-03-20 12:00:00
author:     "Chris"
header-img: "img/home-bg-art.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Big Data
    - Hadoop
---

> Key Words: Hadoop, NFS, Distributed File System

## 为什么需要文件系统间的映射

hdfs是分布式系统，要想访问hdfs上的文件，可以用java api或者hadoop shell等工具。遗憾的是，很多应用包无法直接调用这些api，因此，如果想操作hdfs文件系统就像操作本地文件系统一样的便捷，实现两者之间的映射是我们想到的最好的解决方案。

![](/img/2017-03-20-hadoop-dfs-nfs-lfs/architecture.png)


## 已有解决方案介绍

目前已有解决方案有三种。其中，《Hadoop权威指南》一书中提到，可以通过Fuse－DFS和WebDEV两种途径实现，如下所示：

![](/img/2017-03-20-hadoop-dfs-nfs-lfs/fuse2.png)

遗憾的是，楼长在部署时发现上述方式过于繁琐。相较之下，第三种解决方案：NFS挂载，则更为实用。


## 基于NFS的解决方案实践

为通过NFS技术实现HDFS到本地的挂载，进而完成映射，我们分两步走。

一. 本地目录到本地目录的挂载

首先需要安装nfs服务

```
$sudo apt-get install nfs-kernel-server
$sudo apt-get install nfs-client
```

安装完成后，更新/etc/exports文件，并在最后添加如下一行：

```
/ *(insecure, rw, async, no_root_squash)
```

其中，“／”表示准备被挂载的目录， “＊”对应允许挂载的IP地址，这里表示所有。

安装完成后，启动nfs服务：

```
$sudo /etc/init.d/rpcbind start
$sudo /etc/init.d/nfs-kernel-server start
```

最后，实现挂载。

```
$sudo mount 192.168.0.123:/input /mnt
```

二. HDFS到本地目录的挂载

这一部分是实际操作中bug频出的地方，归根结底是配置文件的错误。对core-site.xml和hdfs-site.xml两大文件，分别添加如下内容：

-- core-site.xml

```
<property>
<name>hadoop.proxyuser.root.groups</name>
<value>*</value>
</property>

<property>
<name>hadoop.proxyuser.root.hosts</name>
<value>192.168.0.123</value>
</property>

```

注意，这里<name>标签里的“root”，在很多教程里写成“nfsserver”或"hadoop"，在楼长的实际操作中，只有“root”不报错。

-- hdfs-site.xml

```
<property>
<name>dfs.namenode.accesstime.precision</name>
<value>3600000</value>
</property>

<property>
<name>nfs.dump.dir</name>
<value>/tmp/.hdfs-nfs</value>
</property>

<property>
<name>nfs.exports.allowed.hosts</name>
<value>* rw</value>
</property>

```

配置文件更新后，同步到各节点，并启动集群。

```
$sbin/start-all.sh
```

使用以下命令启动hadoop中的nfs服务：

```
$sudo /etc/init.d/nfs-kernel-server stop
$sudo /etc/init.d/rpcbind stop
$sudo ./hadoop-daemon.sh start portmap
$sudo ./hadoop-daemon.sh start nfs3
```

查看是否成功：

```
$sudo rpcinfo -p 192.168.0.123
$sudo showmount -e 192.168.0.123
```

服务启起来之后，实现挂载：

```
$sudo mount -t nfs -o vers=3,proto=tcp,nolock,noacl 192.168.0.123:/ /mnt
```




---


