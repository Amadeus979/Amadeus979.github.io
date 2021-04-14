---
layout:     post
title:      Hadoop
subtitle:   Hadoop的分布式文件系统hdfs搭建与Hadoop的配置（单一节点担任多角色的伪分布式模式搭建过程）
date:       2020-04-14
author:     BY, Amadeus979
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Blog

---



[TOC]



# Hadoop的分布式文件系统hdfs搭建

18373046 徐睿远

### 官网

http://hadoop.apache.org/docs/r2.6.5/

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201173402641.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201173525642.png)

#### 平台系统

linux

#### 语言

java

#### 所需软件

##### jdk1.8

java移动性好，编译一次多次执行

##### ssh

以一个用户名@登录到一台主机，ssh是上层通信协议它是7层的，它底层还是使用的tcp这种4层的通信协议

，3层的这种ip这种网络寻址方式。

xshell

多个客户端终端

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201175318451.png)

hadoop hdfs是一个集群，有很多机器，又知道hdfs架构中有多种角色，name node，data node，又知道这些角色都是jvm进程，每台机器这些进程都是怎么启动的？

可以ssh手动把每一台启动起来——过于低效。

Hadoop部署分发的软件包含一些脚本可以代替手动启动过程。

登录都要输入密码——使用基于ssh的免密方式启动。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201175340452.png)

为了让远程管理启动成功，得设置免密。

远程执行有一个弊端，A节点想要启动B节点的一个jvm进程，A节点需要用到B节点的java命令，但是脚本远程执行时不会加载对方的环境变量/etc/profile文件

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201175811954.png)

`vi /etc/profile `：最后加上`export BIGDATA=hello`

保存退出

系统中已经有这个文件了但是bash已经运行

此时运行`echo $BIGDATA`打印不出任何值

需要source命令，使得当前命令行程序重新加载`source /etc/profile`

再echo即可成功

node03里：

`ssh root@192.168.150.14 'mkdir /haha'`

14的这台机器于是就创建了haha这个目录

` cd /`，`ll`

但是13这台机器上去`ssh root@192.168.150.14 'echo $BIGDATA'`

输入密码，就打印不出来

ssh远程登陆过去的时候不会加载对方的/etc/profile文件

解决：

`ssh root@192.168.150.14 'source /etc/profile	;	echo $BIGDATA'`

再输入密码即可获取

ssh拿不到别的机器的java_home

hadoop的配置命令里面要重新配置一遍java_home的绝对路径，要把java安装这件事即告诉操作系统，也告诉hadoop

类似绿色软件，解压即可运行

### 三种模式：

#### Local（Standalone）Mode

单个java进程包含了一些角色，一般只用于debug

#### Pseudo-Distributed Mode

一台机器中不同角色是不同的java进程（注意不是线程）

#### Fully-Distributed Mode

角色分布在集群不同的物理机上，name node,data node分到不同的机器上

说白了就是不同角色在哪里启动最终的一个结果，所有角色都在一台机器上启动就是伪分布式，所有角色分开启动不同角色相互协作就是完全分布式。

生产环境机器多一些，开发环境机器少一些。

未来肯定使用脚本启动，肯定使用`$ sbin/start-dfs.sh`脚本（hadoop script的一个服务脚本，肯定会用到远程跳过去执行命令，把别人的东西启动起来，执行时需要免密）

### 执行

#### 先格式化文件系统

namenode启动必须加载一个fsimage文件来恢复内存，因此另外有个语义是初始化，第一次搭建时也会生成一个空的fsimage，启动时可以加载一个空的内存。第一次搭建集群时才会执行。



## 基础设施

操作系统、环境、网络、必须软件

1.设置IP及主机名

2.关闭防火墙&selinux

3.设置hosts映射

4.时间同步

5.安装jdk

6.设置SSH免秘钥

### 设置网络

#### 设置ip

```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

```shell
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.229.11
NETMASK=255.255.255.0
GATEWAY=192.168.229.2
DNS1=223.5.5.5
DNS2=114.114.114.114

```

### 设置主机名

```shell
vi /etc/sysconfig/network
```

```
NETWORKING=yes
HOSTNAME=node01
```

## 设置本机的ip到主机名的映射关系

```shell
vi /etc/hosts
```

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.229.11 node01
192.168.229.12 node02
192.168.229.13 node03
192.168.229.14 node04
```

## 关闭防火墙

```shell
service iptables stop
chkconfig iptables off
```

## 关闭 selinux

```shell
vi /etc/selinux/config
```

```
SELINUX=disabled
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201233711822.png)

## 做时间同步

```shell
vi /etc/ntp.conf
	server ntp1.aliyun.com
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201233909155.png)

```shell
service ntpd start
```

```shell
chkconfig ntpd on
```

xftp传输

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201234759915.png)

## 安装jdk

```shell
rpm -i   jdk-8u181-linux-x64.rpm
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201234847071.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201234923704.png)

有些软件不认java_home，必须要有/usr/java/default

设置java_home

```shell
vi /etc/profile     
	export  JAVA_HOME=/usr/java/default
	export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile   |  .    /etc/profile
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201235308407.png)

## 免密

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201235507432.png)

自己登自己还需要密码，因此尚未免密

```shell
ssh  localhost
```

1,验证自己还没免密  

2,被动生成了  /root/.ssh

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201201235815334.png)

1想登陆2，1得生成自己的公钥和秘钥，1得把公钥发给对方，对方持有公钥则可免密登陆

```shell
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202000107472.png)

目前是伪分布式，单机中要发出所有这些角色，但是也想使用脚本代替手工启动这些角色。想使用hadoop的脚本就必须使用免密。

免密有两个场景可以用，一个是挑出一台机器，把他的公钥分发给它想管理的所有人，以保证这台机器可以执行管理脚本，和角色没有关系。

```shell
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202000450505.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202000620581.png)

# Hadoop的配置（单一节点担任多角色的伪分布式模式搭建过程）

伪分布式: (单一节点)

1.部署路径

2.配置文件

- hadoop-env.sh

- core-site.xml

- hdfs-site.xml

- slaves

| **host** | **NN** | **SNN** | **DN** |
| -------- | ------ | ------- | ------ |
| node01   | *      | *       | *      |

```shell
mkdir /opt/bigdata
tar xf hadoop-2.6.5.tar.gz
mv hadoop-2.6.5  /opt/bigdata/
pwd
	/opt/bigdata/hadoop-2.6.5

vi /etc/profile	
	export  JAVA_HOME=/usr/java/default
	export HADOOP_HOME=/opt/bigdata/hadoop-2.6.5
	export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
source /etc/profile
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202001414249.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202001538073.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202001604748.png)

sbin目录：service服务，服务脚本命令

bin目录：功能模块，自己的命令

etc：放配置

share：放了一些jar包

lib：放了一些库

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202001906215.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202001951764.png)

成功安装，hdfs已经可以用了

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202002045856.png)

可以看到hadoop的脚本已经有了，部署完成，接下来为配置。

## 配置

只要环境变量是对的：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202002402021.png)

可以开始配置了

JAVA_HOME也要给hadoop配置

## 配置hadoop的角色

```shell
cd   $HADOOP_HOME/etc/hadoop
```

必须给hadoop配置javahome要不ssh过去找不到

```shell
vi hadoop-env.sh
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202002745047.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202002832365.png)

```
export JAVA_HOME=/usr/java/default
```

改图中行，给出NN角色在哪里启动

这个不用source执行

接着执行官网配置

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202002952319.png)

```shell
vi core-site.xml
```

```xml
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node01:9000</value>
    </property>
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202003121401.png)

如图为核心配置

fs.defaultFS值为hdfs://node01:9000，定义了分布式文件系统协议，后面给了主机名加端口号。管理脚本或命令读到了这个文件就知道去哪儿以什么端口来启动一个namenode，如果是datanode这种从角色读到这个配置文件，datanode就知道我应该访问哪一台主机的哪个端口就可以和namenode通信，如果是客户端读到这个配置文件，客户端就知道怎么找到namenode。

直接写死自己主机的名称：node01

```shell
vi hdfs-site.xml
```

配置hdfs  副本数为1（因为是伪分布式）

```xml
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202003905926.png)

配置DN这个角色在哪里启动：

```shell
vi slaves
```

里面放的就是你在哪一台去启动一个datanode

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202004018391.png)

配置太少了，可以展开左侧Configuration

core的：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202004444938.png)

tmp满的时候操作系统会删掉它

hdfs-site.xml：

datanode会把块存成文件存到本地磁盘系统，namenode会把它内存的元数据拍成快照或者日志写到本地的磁盘系统

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202004749579.png)

取到了tmp目录，不可靠。

```xml
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>/var/bigdata/hadoop/local/dfs/name</value>
	</property>
```

(/var/：存数据的)

datanode同理：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202005157533.png)

```xml
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/var/bigdata/hadoop/local/dfs/data</value>
	</property>
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202005355880.png)

配置基本完成

## 初始化&启动：

刚刚的制定的两个目录必须是空目录

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202005535198.png)

现在还没有bigdata目录

先格式化，只格式化namenode，只格式化元数据

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202005806444.png)Saving image创建image并存储到了这个目录下的current...就是已经做完了

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202005945277.png)

现在看已经多了bigdata这个目录

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202010111348.png)

可以看到已经依据我们所想完成了目录创建

但是datanode目录还没有，也就是说格式化只会格式化namenode，不会格式化datanode，它自己运行时会创建

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202010313833.png)

fsimage_0000000000000000000是空的

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202010354360.png)

clusterID：集群id

在一定范围内的物理网络，可互相访问，一个namenode，两个集群两个namenode，每个集群id不一样，B集群的datanode不会访问A集群的namenode。

以上就完成了创建目录并初始化一个空的fsimage的过程。

格式化操作且勿重复操作

重新格式化会重新生成id

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202010810415.png)

去到hadoop的sbin目录

```
vi start-dfs.sh
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202010958068.png)

脚本里会调用启动namenodes，datanodes一系列事情，下面会调用hadoop-deamons.sh，传的参数里面有一个

--script "\$bin/hdfs" start namenode \$nameStartOpt

最核心的命令：\$bin/hdfs

```shell
vi hadoop-deamons.sh
```

最后一行：

```shell
exec "$bin/slaves.sh" --config $HADOOP_CONF_DIR cd "$HADOOP_PREFIX" \; "$bin/hadoop-daemon.sh" --config $HADOOP_CONF_DIR "$@"
```

要执行\$bin/slaves.sh

cd "$HADOOP_PREFIX"：去到安装路径下

再来看slaves.sh

```shell
vi slaves.sh
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202011802135.png)

取出来一个主机，登到你那个主机里面去，取出其所有指令

补充配置secondary：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202012222749.png)

hdfs-site.xml

只有一台：

```shell
	<property>
		<name>dfs.namenode.secondary.http-address</name>
		<value>node01:50090</value>
	</property>
```

second这个角色也是要磁盘路径的

寻找其目录设置：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202012547176.png)

在dfs.namenode.checkpoint.dir

```shell
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>/var/bigdata/hadoop/local/dfs/secondary</value>
	</property>
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202012851398.png)

修改hdfs-site.xml

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202012832635.png)

如果想停止hdfs：`stop-dfs.sh`

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202013200539.png)

## 用ssh免密的方式采用脚本启动dfs

secondary读的是刚刚设置的配置文件，如果没设置好不会显示node01，而会显示四个0

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202013638489.png)

name是格式化得到的，data和secondary是启动的时候得到的。

因此可以看到，第一次启动时datanode和secondary角色会初始化创建自己的数据目录，并且和他们的namenode连接

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202013915653.png)

可以看到也有VERSION文件，可以看到datanode和namenode有相同的CID：CID-bdeabde2-41ab-41bb-ad95-0c0eda4c8653

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202014235065.png)

修改windows： C:\Windows\System32\drivers\etc\hosts

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202014418594.png)

### 验证

在浏览器中输入：

```
http://node01:50070
```

得到：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202014523216.png)

namenode存了元数据，该页面告诉我们namenode实在node01:9000，这个和我们的配置一样，下面的集群id也是和上面所示相同的，active表示是活动的

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202014751642.png)

安全模式已经退出了

Live Nodes：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202014842272.png)

存活的datanode，由于是伪分布式，所以只有它一个。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202014931782.png)

该页面显示了启动过程：

阶段里面第一个是**Loading fsimage /var/bigdata/hadoop/local/dfs/name/current/fsimage_0000000000000000000 321 B**

先加载内存，

然后**Loading edits**，加载配置日志到内存进行合并。

合完之后**Saving checkpoint**滚动我们合并的这个东西。合并完之后就退出了。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202015329399.png)

正在打开的日志：0003尾缀的那个日志。

增量日志（editlog）的id永远是最后一个fsimage的之后的一个数字

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202015724009.png)

secondary启动之后会拿已存的一个fsimage和它增量的一个日志，从namenode拷了一个0000，只需要拿edit数字大于0000 的开始拷就可以，拷完之后会合并成一个新的。

到浏览器，键入

```
http://node01:50090/
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202020011810.png)

只有一个简单的显示，显示了chackpoint的位置：

- file:/var/bigdata/hadoop/local/dfs/secondary

得出观察结论：

cd   /var/bigdata/hadoop/local/dfs/name/current

观察 editlog的id在fsimage的后边

cd /var/bigdata/hadoop/local/dfs/secondary/current

SNN 只需要从NN拷贝最后时点的FSimage和增量的Editlog

### 简单使用

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202020300949.png)

可以看到namenode的目录树还是空的

通过

```shell
hdfs dfs
```

可以知道它支持的关于文件系统的操作有哪些。

```shell
Usage: hadoop fs [generic options]
	[-appendToFile <localsrc> ... <dst>]
	[-cat [-ignoreCrc] <src> ...]
	[-checksum <src> ...]
	[-chgrp [-R] GROUP PATH...]
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
	[-chown [-R] [OWNER][:[GROUP]] PATH...]
	[-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
	[-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-count [-q] [-h] <path> ...]
	[-cp [-f] [-p | -p[topax]] <src> ... <dst>]
	[-createSnapshot <snapshotDir> [<snapshotName>]]
	[-deleteSnapshot <snapshotDir> <snapshotName>]
	[-df [-h] [<path> ...]]
	[-du [-s] [-h] <path> ...]
	[-expunge]
	[-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-getfacl [-R] <path>]
	[-getfattr [-R] {-n name | -d} [-e en] <path>]
	[-getmerge [-nl] <src> <localdst>]
	[-help [cmd ...]]
	[-ls [-d] [-h] [-R] [<path> ...]]
	[-mkdir [-p] <path> ...]
	[-moveFromLocal <localsrc> ... <dst>]
	[-moveToLocal <src> <localdst>]
	[-mv <src> ... <dst>]
	[-put [-f] [-p] [-l] <localsrc> ... <dst>]
	[-renameSnapshot <snapshotDir> <oldName> <newName>]
	[-rm [-f] [-r|-R] [-skipTrash] <src> ...]
	[-rmdir [--ignore-fail-on-non-empty] <dir> ...]
	[-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
	[-setfattr {-n name [-v value] | -x name} <path>]
	[-setrep [-R] [-w] <rep> <path> ...]
	[-stat [format] <path> ...]
	[-tail [-f] <file>]
	[-test -[defsz] <path>]
	[-text [-ignoreCrc] <src> ...]
	[-touchz <path> ...]
	[-usage [cmd ...]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|resourcemanager:port>    specify a ResourceManager
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]

```

可以看到很多选项与linux文件系统类似

```shell
hdfs dfs -mkdir /bigdata
hdfs dfs -mkdir  -p  /user/root
```

使用上述命令

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202020900610.png)

本地并没有显示，而hadoop的hdfs系统已经有显示

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202020935874.png)

第二条命令为一般会创建的目录，作为hdfs中的家目录。



还能往上上传下载数据：

```shell
hdfs dfs -put hadoop*.tar.gz  /user/root
```

把hadoop的部署包上传到hdfs指定的目录

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202021317694.png)

可以看到已经上传了一个块

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202021408005.png)

块的默认大小为128MB，因此该文件会被切分为两个块。

（该文件大小为175.09MB）

文件实际路径如下：

```shell
/var/bigdata/hadoop/local/dfs/data/current/BP-2135574444-192.168.229.11-1606841786988/current/finalized/subdir0/subdir0
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202021849679.png)

#### 验证其是否线性切割存储：

~下：

```shell
for i in `seq 100000`;do  echo "hello hadoop $i"  >>  data.txt  ;done
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202022144868.png)

data.txt文件很小但是超过了1MB。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202022242371.png)

准备将其改成1MB，

```shell
hdfs dfs -D dfs.blocksize=1048576  -put  data.txt
```

以一块大小为1MB单位上传data.txt，现在是root用户，默认上传到/user/root。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202022518323.png)

传完了，副本一个，块大小为1MB。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202022556989.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202022636059.png)

第一块为1MB大小。

去看其在本地的目录：

```shell
/var/bigdata/hadoop/local/dfs/data/current/BP-2135574444-192.168.229.11-1606841786988/current/finalized/subdir0/subdir0
```

```shell
cat /root/data.txt
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202023012047.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202023110843.png)

看到最后一行出现：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202023131808.png)

数据被截断。剩下的在另一个包里。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202023258817.png)