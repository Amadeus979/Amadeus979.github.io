---
layout:     post
title:      Hadoop
subtitle:   完整版分布式HDFS集群搭建
date:       2020-04-14
author:     BY, Amadeus979
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Blog

---



[TOC]

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202103404097.png)

# 完整版分布式HDFS集群搭建

上面已经成功启动一个伪分布式，在一个节点启动了所有角色：NN，DN，SNN

下面准备启动完全分布式。

伪分布式的启动过程可以总结如下：

基础环境

部署配置

1）角色在哪里启动

​	NN： core-site.xml:  fs.defaultFS  hdfs://node01:9000

​	DN:  slaves:  node01

​	SNN: hdfs-siet.xml:  dfs.namenode.secondary.http.address node01:50090

2) 角色启动时的细节配置：

​	dfs.namenode.name.dir

​	dfs.datanode.data.dir

初始化&启动

格式化

​	Fsimage

​	VERSION

start-dfs.sh

​	加载我们的配置文件

​	通过ssh 免密的方式去启动相应的角色

## 从伪分布式到完全分布式：角色重新规划

之前伪分布式：

`jps`：只显示java进程。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202103026276.png)

之前在node01上启动了四个节点。

完全分布式的规划如下：

| **host** | **NN** | **SNN** | **DN** |
| -------- | ------ | ------- | ------ |
| node01   | *      |         |        |
| node02   |        | *       | *      |
| node03   |        |         | *      |
| node04   |        |         | *      |

所有在第一台部署的资料要分发给后面三台。上面图中所示的进程要分配到其他机器上去。

先在node01上将所有角色全部停止：

```
stop-dfs.sh
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202103643437.png)

先用ipconfig检查四台主机的网络情况：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202103928687.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202103954372.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202104007742.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202104020250.png)

他们之间是可以相互ping通的：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202104143405.png)

这需要在node01上做好配置：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202104311312.png)

剩下的就是免密与jdk。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202104505255.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202104533829.png)

只有node01配置了java，node02，node03，node04没配置java接下来做分发。第一台有一个jdk可以分发给后面四台。

```
scp ./jdk-8u181-linux-x64.rpm node02:/root/
scp ./jdk-8u181-linux-x64.rpm node03:/root/
scp ./jdk-8u181-linux-x64.rpm node04:/root/
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202105142275.png)

剩下三台只需要安装即可：

```
rpm -i jdk-8u181-linux-x64.rpm 
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202105438352.png)

第一台的

```
cat /etc/profile
```

最后三行：

```
export JAVA_HOME=/usr/java/default
export HADOOP_HOME=/opt/bigdata/hadoop-2.6.5
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

复制到node02，node03，node04之下的对应文件最后：

```
vi /etc/profile
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202105919292.png)

```
. /etc/profile
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202110341433.png)

第二台可以用了，第三台与第四台同理。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202110712450.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202110724841.png)

接下来是ssh免密：

ssh 免密是为了什么 ：  启动start-dfs.sh：  在哪里启动，那台就要对别人公开自己的公钥。这一台有没有特殊要求，与角色无关联。

可以用任意一台，由于第一台已经做过免密，就用第一台分发其公钥。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202111108869.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202111156784.png)

已经生成过秘钥和公钥，第一台将其公钥分发给2、3、4，就可以免密登陆2、3、4了。

但是2、3、4也需要有如下目录

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202111316781.png)

但这个目录最好不要手动创建——其权限极其苛刻。如果更改了其（第一台）权限，远程登陆是会失效的。登陆过程中会验证这个目录的权限。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202111500245.png)

node02还没有.ssh目录，通过`ssh localhost`生成该目录。

![image-20201202111803806](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201202111803806.png)

该目录下目前只有known_hosts。

对node03、node04也做相同事情。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202111930982.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202111941678.png)

每台都有这个目录后，可以把第一台的公钥分发过去，并把第一台的公钥分加到对方的authorized_keys文件中。

在node01中：

```
scp /root/.ssh/id_dsa.pub  node02:/root/.ssh/node01.pub
scp /root/.ssh/id_dsa.pub  node03:/root/.ssh/node01.pub
scp /root/.ssh/id_dsa.pub  node04:/root/.ssh/node01.pub
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202112224121.png)

在node02中：

```
cd ~/.ssh
cat node01.pub >> authorized_keys
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202112415186.png)

一旦有这个字符串了，第一台就可以免密登陆过去了。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202112530725.png)

再给node03分发过去：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202112711902.png)

在node03中：

```
cd ~/.ssh
cat node01.pub >> authorized_keys
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202112809017.png)

于是在01中可以免密登陆03

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202112834103.png)

04同理。

01的过程：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202113034778.png)

角色配置、资源分发

只需要在第一台部署，然后分发就可以

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202113638298.png)

这是伪分布式的配置，保留火种因此将其复制一份。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202113844844.png)

脚本会对着hadoop目录读文件。即`/opt/bigdata/hadoop-2.6.5/etc`

注意到之前jdk有两次配置，一次配给/etc/profile，一次配给了hadoop-env。这都配过了。

因为规划在node01启动NN，因此core-site.xml不用动。

```
cd $HADOOP/etc/hadoop
vi core-site.xml    不需要改
vi hdfs-site.xml
```

```
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/var/bigdata/hadoop/full/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/var/bigdata/hadoop/full/dfs/data</value>
        </property>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>node02:50090</value>
        </property>
        <property>
                <name>dfs.namenode.checkpoint.dir</name>
                <value>/var/bigdata/hadoop/full/dfs/secondary</value>

```

local已经被写过，换一个目录，设为full

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202115631165.png)

有了namenode，有了secondarynode，现在搞datanode

```
vi slaves
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202114829433.png)

```
node02
node03
node04
```

到部署目录：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202115106700.png)

/opt/bigdata里面有hadoop。

但是node02的/opt里面连bigdata都没有：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202115213323.png)

因此从node01，向node02、node03、node04分发目录。

```
cd /opt
scp -r ./bigdata/  node02:`pwd`
scp -r ./bigdata/  node03:`pwd`
scp -r ./bigdata/  node04:`pwd`
```

node02上已经有了：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202115437661.png)

node03和04同理。

## 格式化启动

来到node01的如下目录：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202130227706.png)

node02等是没有/var/bigdata目录的。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202130044074.png)

在node01上述目录中执行：

```
hdfs namenode -format
```

注意successfully foromatted

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202130351899.png)

观察到full目录已经生成。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202130545260.png)

相应目录下fsimage和VERSION已经生成。

同时观察到dfs下没有secondary和datanode的目录

去node03上观察

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202130647596.png)

没有bigdata目录，因为未来只会启动datanode不会启动namenode。

### 启动

在node01上：

```
start-dfs.sh
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202130924148.png)

这个脚本会读取配置文件，会跳到不同的节点启动不同的角色。跳到第一个节点启动namenode，然后受slaves文件影响去02,03,04上区启动datanode，然后会去02上启动secondarynamenode（受hdfs-site.xml配置）。

然后到node02查看，发现如下图，var下已有bigdata出现

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134010449.png)

node03和node04同理，如果没有的话可能是前面某个配置node01的操作没有在node02上做，停止hadoop，配置做全重启即可。

在第三个节点上查看：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134345173.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134422094.png)

集群ID相同，表示他们之间可以相互建立连接。

node02上：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134548910.png)

有datanode和secondarynamenode的角色。

用jps验证：

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134655612.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134708633.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134719915.png)

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202134732960.png)

第一台只有namenode，第二台有secondarynamenode和datanode，第三和第四台只有datanode。

## 验证

到谷歌浏览器上：

```
http://node01:50070/
```

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202135015544.png)

已经active。

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/image-20201202135047728.png)

datanode已经有了三台。

格式化只会格式化namenode，secondarynamenode不会管，secondarynamenode什么时候启动都可以，它一旦启动就会把它没有的fsimage和editlog拉过来合并。

secondarynamenode没有启动，namenode自己并不一定合并fsimage和editlog，namenode在运行启动后不会做，只会一直写editlog，只有在集群重启的时候namenode启动流程时会加载fsimage和增量的editlog，做一次内存合并之后会在提供对外服务之前会把合并完的内存状态写一个新的fsimage。namenode只会在启动时做一次。所以需要secondarynamenode去做这个合并的事情，因为其消耗IO。

