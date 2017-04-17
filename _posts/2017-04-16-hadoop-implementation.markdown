---
layout:     post
title:      "「51CTO」Hadoop平台与开发环境搭建"
subtitle:   "If you can quit, then quit. If you cannot quit, your are a writer. - R.A.Salvatore"
date:       2017-04-16 12:00:00
author:     "Chris"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Big Data
    - MapReduce
---

> 文章转自我在 51CTO 的一篇网络课程，如下是楼长 2014 年录的一段操作视频，可供大家参考。（优酷的广告可真长...）

<embed src='http://player.youku.com/player.php/sid/XMjcxMjg2MjU2OA==/v.swf' allowFullScreen='true' quality='high' width='690' height='320' align='middle' allowScriptAccess='always' type='application/x-shockwave-flash'/>

## 准备工作

1.&nbsp;集群安装 ubuntu 系统，用户名：hadoop，密码自设。

2.&nbsp;使用 <b>$ifconfig</b> 命令查看网络信息，并据此将 IP 等设置为静态。集群相应网络参数如下表，这里我们使用了三台 Linux 来部署Hadoop。

| Host Name     | User Name  | IP Address |
| ------------- |------------| -----------|
| Master        | hadoop     | 10.0.1.27  |
| Slave_one     | hadoop     | 10.0.1.27  |
| Slave_two     | hadoop     | 10.0.1.27  |

3.&nbsp;实现 ssh 服务，命令：<b>$sudo apt-get install openssh-server</b>（不需要重启）。

4.&nbsp;使用命令：<b>$ssh hadoop@10.0.1.27</b> 测试服务连接是否正常。

5.&nbsp;设置无密钥登录

各节点生成密钥：<b>$ssh-keygen</b>，之后会在 /home/hadoop/ 路径下生成 .ssh 文件夹；

将各节点的 id_rsa.pub 文件集中到 Master：

<b>$scp ~/.ssh/id_rsa.pub hadoop@10.0.1.27:/home/hadoop/keygen/</b>

将各节点 id_rsa.pub 中的内容追加到authorized_keys文件：

<b>$cat id_rsa.pub >> ../.ssh/authorized_keys</b>

随后，将 authorized_keys 文件分发到各个节点：

<b>$scp ~/.ssh/authorized_keys hadoop@10.0.1.27:/home/hadoop/.ssh/
</b>

如此,就实现了各节点间的无密钥登录。注意:无密钥登录是集群正常运行的必要条件

6.&nbsp;设置 hostname (Master, Slave_one, Slave_two),执行命令：<b>$sudo vim /etc/hosts</b>

7.&nbsp;手动配置 JDK

为了进行版本控制，这里我们手动安装 JDK。将文件夹 jdk1.7.0-40 上传到服务器各节点：

<b>$./auto_sync_simple.sh jdk1.7.0_40 /home/hadoop/Cloud/</b>

这里，auto_sync_simple.sh 是自己写的分发脚本，大家也可以命令行直接分发。

接着，打开 /etc/profile 文件，追加如下信息：

```jason
export JAVA_HOME=/home/hadoop/Cloud/jdk1.7.0_40
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib 
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOME/bin
```

执行命令<b>$. /etc/profile</b> 使环境变量即时生效

每个节点均如此设置,可使用 <b>$java -version</b> 命令进行验证

8.&nbsp;关闭防火墙

ubuntu 系统默认 iptables 是关闭的，通过命令 <b>$sudo ufw status</b> 可查看状态 

关闭 SELinux: <b>$setenforce 0</b>

## 部署 Hadoop

1.&nbsp;Hadoop目录结构

Hadoop-2.0 以后版本较之前有较大改动

由于使用 hadoop 的用户被分成了不同的用户组,就像 Linux 一样。因此执行文件和脚本被分成了
两部分,分别存放在 bin 和 sbin 目录下。

存放在 sbin 目录下的是只有超级用户才有权限执行的脚本, 如 start-dfs.sh, start-yarn.sh, stop-dfs.sh, stop-yarn.sh 等,这些是对整个集群的操作,只有 superuser 才有权限。

存放在 bin 目录下的脚本所有的用户都有执行的权限,这里的脚本一般都是 对集群中具体的文件或者 block pool 操作的命令,如上传文件,查看集群的使用情况等

etc 目录下存放的就是在 0.23.0 版本以前 conf 目录下存放的东西,就是对 common, hdfs, mapreduce(yarn)的配置信息

include 和 lib 目录下存放的是使用 Hadoop 的 C 语言接口开发用到的头文件和链接的库

libexec 目录下存放的是 hadoop 的配置脚本,具体怎么用到的这些脚本,我也还没跟踪到。目前 我就是在其中 hadoop-config.sh 文件中增加了 JAVA_HOME 环境变量

logs 目录在 download 到的安装包里是没有的,如果你安装并运行了 hadoop,就会生成 logs 这 个目录和里面的日志

share 这个文件夹存放的是 doc 文档和最重要的 Hadoop 源代码编译生成的 jar 包文件,就是运行 hadoop 所用到的所有的 jar 包

2.&nbsp;从 Apache 官网可以下载最新版，本文采用的是 2.2.0，<http://hadoop.apache.org/ >

3.&nbsp;在服务器各节点新建 Cloud 目录，注意，各节点 Hadoop 保存的路径需要一致。

4.&nbsp;将从官网下载的 hadoop-2.2.0.tar.gz 上传到服务器

<b>$scp Downloads/hadoop-2.2.0.tar.gz hadoop@10.0.1.27:/home/hadoop/Cloud/</b>

解压文件：<b>$tar -zxvf hadoop-2.2.0.tar.gz hadoop-2.2.0</b>

5.&nbsp;如下文件需要配置

*core-site.xml, *mapred-site.xml, *hdfs-site.xml, *yarn-site.xml, *masters, *slaves

6.&nbsp;编辑~/.bashrc 文件，加入如下内容

```jason
export HADOOP_PREFIX="/home/hadoop/Cloud/hadoop-2.2.0" 
export PATH=$PATH:$HADOOP_PREFIX/bin
export PATH=$PATH:$HADOOP_PREFIX/sbin
export HADOOP_MAPRED_HOME=${HADOOP_PREFIX} 
export HADOOP_COMMON_HOME=${HADOOP_PREFIX} 
export HADOOP_HDFS_HOME=${HADOOP_PREFIX}
export YARN_HOME=${HADOOP_PREFIX}
```

保存退出，然后执行<b>$source ~/.bashrc</b> 使之即时生效

7.&nbsp;在 etc/hadoop 目录中依次编辑如上所述配置文件，若未找到 mapred-site.xml 文件可自行创建，其中 core-site.xml、mapred-site.xml、hdfs-site.xml、yarn-site.xml 为配置文件，需要着重配置，各文件配置内容如下所示:

<b>(1) core-site.xml</b>

```html
<property> 
    <name>io.native.lib.avaliable</name> 
    <value>true</value>
</property>

<property> 
    <name>fs.default.name</name> 
    <value>hdfs://mcmaster:9000</value> 
    <final>true</final>
</property>

<property>
    <name>hadoop.tmp.dir</name> 
    <value>/home/hadoop/Cloud/workspace/tmp</value>
</property>
```

<b>(2) hdfs-site.xml</b>

```html
<property> 
    <name>dfs.replication</name> 
    <value>3</value>
</property>

<property> 
    <name>dfs.permissions</name> 
    <value>false</value>
</property>

<property>
    <name>dfs.namenode.name.dir</name> 
    <value>/home/hadoop/Cloud/workspace/hdfs/data</value> 
    <final>true</final>
</property>

<property>
    <name>dfs.namenode.dir</name> 
    <value>/home/hadoop/Cloud/workspace/hdfs/name</value>
</property>

<property>
    <name>dfs.datanode.dir</name> 
    <value>/home/hadoop/Cloud/workspace/hdfs/data</value>
</property>

<property> 
    <name>dfs.webhdfs.enabled</name> 
    <value>true</value>
</property>
```

<b>(3) mapred-site.xml</b>

```html
<property> 
    <name>mapreduce.framework.name</name> 
    <value>yarn</value>
</property> 

<property>
    <name>mapreduce.job.tracker</name> 
    <value>hdfs://mcmaster:9001</value> 
    <final>true</final>
</property> 

<property>
    <name>mapreduce.map.memory.mb</name>
    <value>1536</value> 
</property>

<property> 
    <name>mapreduce.map.java.opts</name> 
    <value>-Xmx1024M</value>
</property> 

<property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>3072</value> 
</property>

<property> 
    <name>mapreduce.reduce.java.opts</name> 
    <value>-Xmx2560M</value>
</property>

<property> 
    <name>mapreduce.task.io.sort.mb</name> 
    <value>512</value>
</property> 

<property>
    <name>mapreduce.task.io.sort.factor</name>
    <value>100</value> 
</property>

<property> 
    <name>mapreduce.reduce.shuffle.parallelcopies</name> 
    <value>50</value>
</property> 

<property>
    <name>mapred.system.dir</name>
    <value>/home/hadoop/Cloud/workspace/mapred/system</value> 
    <final>true</final>
</property> 

<property>
    <name>mapred.local.dir</name>
    <value>/home/hadoop/Cloud/workspace/mapred/local</value> 
    <final>true</final>
</property>
```

<b>(4) yarn-site.xml</b>

```html
<property> 
    <name>yarn.resourcemanager.address</name> 
    <value>mcmaster:8080</value>
</property>

<property> 
    <name>yarn.resourcemanager.scheduler.address</name> 
    <value>mcmaster:8081</value>
</property>

<property> 
    <name>yarn.resourcemanager.resource-tracker.address</name> 
    <value>mcmaster:8082</value>
</property>

<property> 
    <name>yarn.nodemanager.aux-services</name> 
    <value>mapreduce_shuffle</value>
</property>

<property> 
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```

8.&nbsp;在 etc/hadoop 目录下的 hadoop-env.sh 中添加如下内容，另需 yarn-env.sh 中填充相同的内容。
```html
export HADOOP_FREFIX=/home/hadoop/Cloud/hadoop-2.2.0 
export HADOOP_COMMON_HOME=${HADOOP_FREFIX} 
export HADOOP_HDFS_HOME=${HADOOP_FREFIX}
export PATH=$PATH:$HADOOP_FREFIX/bin
export PATH=$PATH:$HADOOP_FREFIX/sbin
export HADOOP_MAPRED_HOME=${HADOOP_FREFIX}
export YARN_HOME=${HADOOP_FREFIX}
export HADOOP_CONF_HOME=${HADOOP_FREFIX}/etc/hadoop 
export YARN_CONF_DIR=${HADOOP_FREFIX}/etc/hadoop 
export JAVA_HOME=/usr/lib/jvm/java-7-sun
```

9.&nbsp;编写分发脚本，将配置完成的 hadoop 分发的所有节点

<b>$./auto_sync_simple.sh /home/hadoop/Cloud/hadoop-2.2.0 /home/hadoop/Cloud/</b>

10.&nbsp;跳转到 hadoop 根目录下，格式化 namenode，随后启动集群

<b>$bin/hdfs namenode -format sbin/start-all.sh</b>

11.&nbsp;登录 10.0.1.27:8080 可查看资源管理页面

![](/img/2017-04-16-hadoop-implementation/hadoop-login.png)


## 运行示例程序

WordCount 是最简单也是最能体现 MapReduce 思想的程序之一，可以称为 MapReduce 版 "Hello World"，该程序的完整代码可以在 Hadoop 安装包的 "src/examples" 目录下找到。

WordCount主要功能是：统计一系列文本文件中每个单词出现的次数。

1.&nbsp;创建本地示例文件

在/home/hadoop/Cloud/hadoop-2.2.0 目录下创建示例文件 test1.txt 和 test2.txt

2.&nbsp;在 HDFS 上创建输入文件 <b>$bin/hadoop fs -mkdir /input</b>

3.&nbsp;上传本地文件到 HDFS 的 input 目录

<b>$bin/hadoop fs -put test1.txt /input</b>

<b>$bin/hadoop fs -put test2.txt /input</b>

4.&nbsp;运行 WordCount

<b>$bin/hadoop jar /share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount
/input /output</b>

若正常执行则说明集群搭建成功!

## 常见问题及解决方案

1.&nbsp;日志报错：

*java.lang.IllegalArgumentException: The ServiceName: mapreduce.shuffle set in
yarn.nodemanager.aux-services is invalid.The valid service name should only contain a-zA-Z0-9*

解决方法：新版本 Hadoop 的配置名有所修改，将如下配置

```html
<property>
    <name> yarn.nodemanager.aux-services </name>
    <value> mapreduce.shuffle </value>
</property>
```

改为

```html
<property>
    <name> yarn.nodemanager.aux-services </name>
    <value> mapreduce_shuffle </value>
</property>
```

2.&nbsp;日志报错：

*Hadoop 2.2.0 - warning: You have loaded library /home/hadoop/2.2.0/lib/native/libhadoop.so.1.0.0
which might have disabled stack guard*

解决方法: 由于 Hadoop 2.2.0 默认配置的 libhadoop 是32位的，在64位的操作系统环境运行过程中，会提示如上错误
需要重新编译 Hadoop 源码,得到适合的库文件,请按以下步骤执行。

(1) 配置编译环境

本文采用 Ubuntu 14.04 系统，首先安装编译环境

<b>$ sudo apt-get install build-essential</b>

<b>$ sudo apt-get install g++ autoconf automake libtool cmake zlib1g-dev pkg- config libssl-dev</b>

<b>$ sudo apt-get install maven</b>

(2) 配置 protobuf

编译过程需要使用 protobuf，建议先行安装。截至撰文，Ubuntu 仓库默认的 protobuf 是 2.4.1 版,需要更新的
2.5 版。可从 <https://github.com/google/protobuf> 下载。

<b>$ ./configure --prefix=/usr</b>

<b>$make</b>

<b>$sudo make install</b>

可通过 <b>echo $?</b> 命令查看是否 make 成功，返回 0 则通过。

(3) 编译 Hadoop

解压进入 hadoop 源码目录,执行编译

<b>$ mvn package -Pdist,native -DskipTests -Dtar</b>

若安装成功，系统会提示如下信息：

```jason
1. [INFO] BUILD SUCCESS
2. [INFO] ------------------------------------------------
3. [INFO] Total time: 15:39.705s
4. [INFO] Finished at: Fri Nov 01 14:36:17 CST 2014
5. [INFO] Final Memory: 135M/422M
```

然后，在以下目录可以获取编译完成的 libhadoop:

```
1. hadoop-2.2.0-src/hadoop-dist/target/hadoop-2.2.0/lib
```

将编译的 lib/native 文件夹替换原本的即可。

注意：在 mvn hadoop-2.4.0 时一切正常，但编译 hadoop-2.2.0 版本时报如下错误

```jason
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] /home/hduser/code/hadoop-2.2.0-src/hadoop-common-project/hadoop-
auth/src/test/java/org/apache/hadoop/security/authentication/client/AuthenticatorTestCase.java:[88
,11] error: cannot access AbstractLifeCycle
[ERROR] class file for org.mortbay.component.AbstractLifeCycle not found
/home/hduser/code/hadoop-2.2.0-src/hadoop-common-project/hadoop-
auth/src/test/java/org/apache/hadoop/security/authentication/client/AuthenticatorTestCase.java:[96
,29] error: cannot access LifeCycle
[ERROR] class file for org.mortbay.component.LifeCycle not found
```

此时，需编辑 hadoop-common-project/hadoop-auth/pom.xml 文件，添加依赖

```html
<dependency>
    <groupId>org.mortbay.jetty</groupId>
    <artifactId>jetty-util</artifactId>
    <scope>test</scope>
</dependency>
```

再次编译，这个错误解决了。

3.&nbsp;错误现象：

```
WARN hdfs.DFSClient: DataStreamer Exception 
org.apache.hadoop.ipc.RemoteException(java.io.IOException):File /tmp/pg20417.txt._COPYING_ could only be replicated to 0 nodes instead ofminReplication (=1). 
There are 0 datanode(s) running and no node(s) are excluded in this operation.
```

发生地方:执行 <b>$bin/hdfs dfs –copyFromLocal /home/hadoop/test1.txt /input</b> 时

原因定位: 后来经过反复查看,是因为 fs.default.name 的值中的 IP 地址配置成了 localhost，导致系统找不到 hdfs，是在 datanode 的日志中发现这个错误的

4.&nbsp;错误现象：

Namenode 正常启动,但子节点的 datanode 无法正常启动

原因定位:Hadoop 文件系统格式不符，将所有子节点中的/home/hadoop/Cloud/workspace 目录删除，然后格式化 hdfs 并重启集群即可。

5.&nbsp;错误现象：

提交任务时卡住,MapReduce 无法正常执行

原因定位:这个问题纠结了两天,网上各种查,最后发现是 yarn-site.xml 文件中参数名: yarn.resoucemanager.scheduler.address 漏写了一个字母,无限捶胸顿足,希望大家引以为戒。

## 开发环境搭建

这里在 Windows 下搭建开发环境，在基于 Ubuntu 的集群上运行，架构如图所示。

![](/img/2017-04-16-hadoop-implementation/hadoop-os.png)

1.&nbsp;安装 JDK 环境

注意，这里安装的 JDK 要和集群上的版本保持一致。下载地址：<http://www.oracle.com/technetwork/java/javase/downloads/index.html>

2.&nbsp;安装 Eclipse

这里楼长用 Eclipse 作为示例，下载地址：<http://www.eclipse.org/downloads/>。感兴趣的童鞋也可以尝试 IntelliJ，一个很不错的 IDE。

3.&nbsp;新建工程

![](/img/2017-04-16-hadoop-implementation/hadoop-new-project.png)

4.&nbsp;新建 package

![](/img/2017-04-16-hadoop-implementation/hadoop-new-package.png)

5.&nbsp;导入 jar 包

![](/img/2017-04-16-hadoop-implementation/hadoop-import.png)

6.&nbsp;编写代码

![](/img/2017-04-16-hadoop-implementation/hadoop-coding.png)

7.&nbsp;编译程序

![](/img/2017-04-16-hadoop-implementation/hadoop-export1.png)

8.&nbsp;导出程序

![](/img/2017-04-16-hadoop-implementation/hadoop-export2.png)

9.&nbsp;将 jar 包复制到 Master 节点，跳转到 Master 的 Hadoop 目录，执行：

<b>$bin/hadoop jar ~/Cloud/WordCount.jar /input /output</b>

以上。

---

