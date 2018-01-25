---
title: hadoop2.7.5集群搭建
copyright: true
date: 2018-01-09 19:40:23
password:
mathjax: true
categories:
  - hadoop
tags:
  - hadoop
  - 搭建
---
### 搭建hadoop2.7.5集群

<blockquote>组内最近要搭建一个`spark`平台，先让我们探探路，于是我就去阿里云和腾讯云各租了一台服务器用来搭建一个`hadoop`集群(-_-)。</blockquote>

<!--more-->

talk is check, action!

#### 环境准备

##### 系统信息

两台服务器的系统都是`cent os 7.3`，其他版本也是大同小异。

##### 创建用户

在每台结点上创建一个名为`hadoop`的用户并修改密码：
```sh
useradd hadoop
passwd hadoop
```

我们的`hadoop`程序都是部署在每台结点的`hadoop`用户上的，所以本文以后的操作如无特别说明，都是在`hadoop`用户上进行。


##### 配置jdk

先安装`jdk`：
```sh
sudo yum install java-1.8.0-openjdk.x86_64
```

再配置环境变量：
```sh
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
```

##### 配置hosts文件

我这两个服务器的外网`ip`和内网`ip`为：

|         | 公网ip     | 内网ip     |
|:-------:|:----------:|:----------:|
| 服务器1 | node1_ip_1 | node1_ip_2 |
| 服务器2 | node2_ip_1 | node2_ip_2 |


在`node1`的`/etc/hosts`文件后面添加下入面内容：
```sh
node1_ip_2  node1    node1
node2_ip_1  node2    node2
```
在`node2`的`/etc/hosts`文件后面加入下面内容：
```sh
node1_ip_1  node1   node1
node2_ip_2  node2   node2
```

可以发现在给`A`结点配置`hosts`文件时，除了`A`结点外，在`hosts`文件内配置的都是外网`ip`，而`A`结点本身的配置只能是内网`ip`，其他结点类似。注意这一点很重要，不然会导致无法启动`namenode`。（很重要，血的教训）

##### 配置免密码登录

配置所有结点间的`ssh`免密码登录。原理就是如果要配置结点`A`免密码登录`B`，那么只需要将`A`结点下`.ssh`目录下的公钥（一般为`id_rsa.pub`文件的你内容）复制到结点`B`的`.ssh/authorized_keys`文件中即可。

这个操作使用`scp-copy-id`命令完成即可。

比如在这里的`node1`需要进行如下配置

```sh
ssh-keygen # 产生秘钥
ssh-copy-id localhost
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@node2
```
结点`node2`是一样的。

##### 打开阿里云服务器的端口

如果是使用的阿里云或者腾讯云，这些服务器的端口默认是关闭的（包括22号端口），要打开的话要登录相应的官网，找到安全组规则，点击快速添加规则，填写下图的内容便打开了所有的端口。

![guize](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/guize.png?raw=true
)

#### hadoop配置

##### 下载hadoop

在所有结点运行：
```sh
 wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.8.1/hadoop-2.8.1.tar.gz -P /tmp
 tar -xf /tmp/hadoop-2.8.1.tar.gz -C /usr/local/hadoop --strip-components 1
```

##### 配置hadoop运行环境

在所有结点的`~/.bash_profile`中加入下面内容：
```sh
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```

执行`source ~/.bash_profile`即可。

##### 修改配置文件

对所有结点的`$HADOOP_HOME/etc/hadoop/`中的配置文件做如下配置：

在`hadoop-env.sh`中加入：
```xml
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
```

向`core-site.xml`添加如下内容：

```xml
<property>
    <name>fs.defaultFS</name>       <value>hdfs://node2:9000/</value>
</property>
<property>
    <name>hadoop.tmp.dir</name>     <value>/home/hadoop/hdpdata</value>
</property>
```
在`hdfs-site.xml`中加入如下内容：
```xml
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///home/hadoop/namenode</value>
</property>
```

在`mapred-site.xml`中加入如下内容：
```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

在`yarn-site.xml`中加入如下内容：
```xml
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>node2</value>
</property>

<property>
    <name>yarn.nodemanager.hostname</name>
    <value>node2</value>
</property>

<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

在`slaves`文件内加入如下内容：
```xml
node1
node2
```

`slaves`内的内容代表`datanode`结点。

注意上面的配置在一台结点配置好了之后一定要将它们发送到所有结点，使得所有结点的`hadoop`配置相同。

##### 格式化namenode

```sh
hdfs dfs namenode -format
```

##### 启动hadoop

```sh
start-dfs.sh
start-yarn.sh
```

输入`jps`，查看显示结果是否各个组件都成功启动：
```
9461 ResourceManager
8999 NameNode
9131 DataNode
9293 SecondaryNameNode
15038 Jps
9582 NodeManager
```
如果某个组件没有成功启动，可以查看`$HADOOP_HOME/logs/`下相应的日志文件。

##### 测试hadoop

```sh
hdfs dfs -mkdir /input
hdfs dfs -copyFromLocal /home/hadoop/words.txt /input/
hdfs dfs -ls /input
hdfs dfs -cat /input/words.txt
```

好了可以愉快的玩耍了。。。

#### 补充网络知识：

##### 服务器公网ip 

可以用于域名解析ip，服务器远程登录ip，是最主要的服务器ip地址。 
    
##### 内网ip 

不能用于域名解析。 


不可以直接用于服务器远程登录，其主要作用是：跟当前帐号下的其他同集群的机器通信。 
　

一些小型企业或者学校，通常都是申请一个固定的IP地址，然后通过IP共享（IP Sharing），使用整个公司或学校的机器都能够访问互联网。而这些企业或学校的机器使用的IP地址就是内网IP，内网IP是在规划IPv4协议时，考虑到IP地址资源可能不足，就专门为内部网设计私有IP地址（或称之为保留地址），一般常用内网IP地址都是这种形式的：10.X.X.X、172.16.X.X-172.31.X.X、192.168.X.X等。需要注意的是，内网的计算机可向Internet上的其他计算机发送连接请求，但Internet上其他的计算机无法向内网的计算机发送连接请求。 
　　

公网IP就是除了保留IP地址以外的IP地址，可以与Internet上的其他计算机随意互相访问。我们通常所说的IP地址，其实就是指的公网 IP。互联网上的每台计算机都有一个独立的IP地址，该IP地址唯一确定互联网上的一台计算机。这里的IP地址就是指的公网IP地址。


其实，互联网上的计算机是通过“公网IP＋内网IP”来唯一确定的，就像很多大楼都是201房间一样，房间号可能一样，但是大楼肯定是唯一的。公网IP地址和内网IP地址也是同样，不同企业或学校的机器可能有相同的内网IP地址，但是他们的公网IP地址肯定不同。那么这些企业或学校的计算机是怎样IP地址共享的呢？这就需要使用NAT（Network Address Translation,网络地址转换）功能。当内部计算机要连接互联网时，首先需要通过NAT技术，将内部计算机数据包中有关IP地址的设置都设成NAT主机的公共IP地址，然后再传送到Internet，虽然内部计算机使用的是私有IP地址，但在连接Internet时，就可以通过NAT主机的NAT技术，将内网我IP地址修改为公网IP地址，如此一来，内网计算机就可以向Internet请求数据了。</br>
                            ——百度百科 
