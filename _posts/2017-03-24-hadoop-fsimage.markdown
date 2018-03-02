---
layout:     post
title:      "Hadoop进阶系列 - 查看HDFS文件物理存储路径的一两点思路"
subtitle:   ""
date:       2017-03-24 12:00:00
author:     "Chris"
header-img: "img/home-bg-art.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Big Data
    - Hadoop
---

> Key Words: Hadoop, NFS, Distributed File System

## 过程总览

hdfs文件均存放在datanode上，namenode上不会存放文件。当客户上传一个文件后，namenode会先对文件作相应的处理（比如按照block大小进行分割）。这里主要讲述存放的一个整体过程以及如何快速的找到存放的节点位置信息。

实现namenode的源码中有一个与文件系统存储和管理有关的关键类FSNameSystem，里面有以下的一些概念：

1. INode: 用来存放文件及目录的基本信息：名称，父节点、修改时间，访问时间以及UGI信息。 
2. INodeFile: 继承自INode，除INode信息外，还有组成这个文件的Blocks列表，重复因子，Block大小 
3. INodeDirectory：继承自INode，此外还有一个INode列表来组成文件或目录树结构 
4. Block(BlockInfo)：组成文件的物理存储，有BlockId，size ，以及时间戳 
5. BlocksMap: 保存数据块到INode和DataNode的映射关系 
6. FSDirectory：保存文件树结构，HDFS整个文件系统是通过FSDirectory来管理 
7. FSImage：保存的是文件系统的目录树 
8. FSEditlog:  文件树上的操作日志 
9. FSNamesystem: HDFS文件系统管理 


### 思路一

Namenode内有两张重要的映射关系表：文件系统的命名空间，文件-block映射表。这对应了两个步骤：1. 文件按照blocksize分割成文件块 2. 文件块和block块的映射。

至此，我们给出一个查找文件存放位置的思路，即首先找到每个文件对应的文件块。具体来说，可以通过 workspce/hdfs/name/current 目录下的edits和fsimage文件查看。

注意，这里的workspace目录即为hdfs-site.xml等配置文件中所指路径。

通过文件块给出的block的id号找到block信息，即可找到对应位置。这中间会找到一个BlockMap，详见参考[[1]]。

current下主要有两大类文件 Edits 和 Fsimage([[2]][[3]]):

- Fsimage(文件系统元数据)：是 Hadoop 文件系统元数据的一个永久性的检查点，其中包含Hadoop文件系统中的所有目录和文件 idnode 的序列化信息

- Edits(编辑日志)：存放的是 Hadoop 文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先会被记录到 edits 文件中

直接vim上面两种文件得到的是相关的序列化信息（总之类似乱码，看不懂）；需要运用 hadoop 提供的相关机制进行查看（参考[[4]][[5]]）。


-- Fsimage

```
$cd ~/workspace/.../current
$hdfs oiv -p XML -i /home/hadoop/cloud/workspace/hdfs/name/current/fsimage_xxx -o fsimage1.xml
$vim fsimage1.xml
```

-- Edits

```
$cd ~/workspace/.../current
$hdfs oev -i /home/hadoop/cloud/workspace/hdfs/name/current/edits_xxx -o edits1.xml
$vim edits.xml
```

相关的fsimage和edits的资料可参见[[6]][[7]]。

在上面提到的datanode下的current下面会有一个BP开头的目录，里面有个current/finalized，这就是存储的各个文件块block。


### 思路二

Hadoop直接提供了查找datanode信息的指令fsck [[8]]

```
$hdfs fsck /input -files -blocks -locations -racks
```

- files 代表检出所有文件状态
- blocks 代表打印文件的block报告
- locations代表打印出文件存放的datanode（分成多少文件块就打印出多少条信息）
- racks代表打印出存放的机架位置


---


[1]: HDFS文件系统如何查看文件对应的block. http://blog.csdn.net/xuefengmiao/article/details/25025455
[2]: Hadoop文件系统元数据fsimage和编辑日志edits：https://www.iteblog.com/archives/968.html
[3]: NameNode中的FSImage文件：https://www.cnblogs.com/miner007/p/3745332.html
[4]: Hadoop-2.4.1学习之edits和fsimage查看器：http://blog.csdn.net/skywalker_only/article/details/40650427
[5]: 查看fsimage和edits：http://blog.csdn.net/baolibin528/article/details/44995541
[6]: hadoop知识之fsimage和editlog：http://blog.csdn.net/maixia24/article/details/38396281
[7]: hdfs里的文件下载HDFS之fsimage、metadata、edits、fstime：https://www.cnblogs.com/zlslch/p/5836961.html
[8]: hdfs fsck命令查看HDFS文件对应的文件块信息(Block)和位置信息：http://lxw1234.com/archives/2015/08/452.htm


