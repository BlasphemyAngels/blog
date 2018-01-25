---
title: linux常用命令（持续添加）
date: 2017-08-03 23:14:21
categories:
  - Linux
tags:
  - linux
  - 命令
---

## 正文
&ensp;&ensp;&ensp;在使用`linux`过程中会学习到很多有用的新命令。本文就总结一些我目前遇到的，以后会持续添加。 
<!--more-->

### xsel

&ensp;&ensp;&ensp;xsel是一个在linux下管理复制粘贴功能的命令，具体可以使用`man xsel`查看。

&ensp;&ensp;&ensp;我使用这个命令的初衷是我在`linux`下工作时。经常需要复制文件名或者文件的内容，当然可以使用鼠标，但是作为`vim`重度使用者，使用鼠标是不是太麻烦了？QAQ，于是就鼓捣了一些，找到了这个命令。

#### 用法

&ensp;&ensp;&ensp;在这里介绍一些常用的用法。

&ensp;&ensp;&ensp;`xsel`是用来管理复制粘贴的选择器的，所谓选择器就是复制的数据放在哪（我的理解)，这里我用到的选择器是系统复制粘贴版`clipboard`。

常用属性：

* 输入选项
  * `-a, --append`
    将标准输入中的内容添加到选择器中
  * `-f, --follow`
    当标准输入增加时，将增加的部分添加到选择器（我也不懂待查，一般也用不着）
  * `-i, --input`
    将标准输入放入选择器
* 输出选项
  * `-o, --output`
    将选择器中的内容放入到标准输出中
* 活动选项
  * `-c, --clear`
    清空选择器中的内容
* 选择器选项
  * `-b, --clipboard`
    指定选择器为clipboard

&ensp;&ensp;&ensp;还有其他一些选项，但是不常用，这里不介绍，如果需要可以查看`man xsel`

常见使用：

* `cat filename | xsel -b -i`

### tr 
&ensp;&ensp;&ensp;`tr`命令用来对标准输入中的字符进行转换和删除，然后将转换后的内容输出到标准输出。 
#### 命令格式

&ensp;&ensp;&ensp;`tr [选项] 字符集1 [字符集2]` 将标准输入中的字符串1的字符转换为字符串2中的字符。

#### 选项

* `-c, -C, --complement`
    使用字符集1时使用的是它的补集，使用这个命令后，这个命令的字符集1已经实际上变成我们制定的字符集的补集。
* `-d, --delete`
    删除字符集1中的字符，这里注意`-d`和`-c`选项搭配时的效果（删除咱们指导的字符集的补集中的字符）。
* `-s, --squeeze-repeats`
    对于最后一个指定的字符集r如果指定了两个字符集那么就是字符集2，如果指定了一个字符集，那么就是字符集1）中的字符，如果它在标准输入中连续出现多次，那么将多次变为一次显示。如标准输入是`adfffffddccd`，指定的最后一个字符集是`fd`那么标准输入会将连续的f和连续的d变为一次，即标准输入变为`adfdccd`
* `-t, --truncate-set1`
    `tr`命令最终的结果是将字符串1中的字符转换为字符串2中的字符，注意如果字符串1的长度和字符串2的长度一样，那么就可以从左到右一一对应转换，但是如果长度不一样该怎么办呢，默认情况下，如果字符集2的长度大于字符集1的长度，超出的部分被忽略，如果字符集1的长度大于字符集2的长度，通过将字符集2的最后一个字符重复多次将字符集2的长度扩展至跟字符集1一样。但是我们不想用这种扩展方式怎么办，这时`-t`就是将长度长的字符集1`截断`跟长度短的字符集2一样的长度。
* `--help` 显示帮助文档。 
* `--version` 显示版本

#### 字符集

* \NNN   八进制按键编码
* \a Ctrl-G  铃声\007
* \b Ctrl-H  退格符\010
* \f Ctrl-L  走行换页\014
* \n Ctrl-J  新行\012
* \r Ctrl-M  回车\015
* \t Ctrl-I  tab键\011
* \v Ctrl-X  \030
* [:alnum:] 所有字母和数字
* [:alpha:] 所有字母
* [:blank:] all horizontal whitespace
* [:cntrl:] 所有控制字符
* [:digit:] 所有数字
* [:graph:] 图形符号
* [:lower:] 所有小写字母
* [:print:] 所有可打印字符
* [:punct:] 所有标点符号
* [:space:] 所有空白符号
* [:upper:] 所有大写字母
* [:xdigit:] 十六进制字符
* [=CHAR=] 所有等于CHAR的字符
* CHAR1-CHAR2 字符1到字符2之间的所有字符。
* [CHAR*] 根据需要将CHAR连续复制多次，达到一定长度
* [CHAR*REPEAT] 连续REPEAT次字符

&ensp;&ensp;&ensp;上面即展示了一些字符在`tr`命令中的书写形式，也表示了`tr`命令中的字符集1和字符集2可以使用的所有字符类别。

#### 注意

&ensp;&ensp;&ensp;如果没有`-d`选项而且字符集1和字符集2都给定了，那么进行转换操作（将字符集1中的字符转换为字符集2中的字符，参见`-t`选项)，而且`-t`选项只会在转换操作时使用。字符集2会在必要的时候通过重复它的最后一个字符扩展到跟字符集1的长度一样（除非指定`-t`命令，这样就会截断字符集1与字符集2的长度一样）。如果字符集2的长度大于字符集1的长度，超过的部分忽略。`-s`选项使用的是指定的最后一个字符集，`-s`选项的执行顺序在转换或删除操作之后。只有`[:lower:]`和`[:upper:]`被保证在扩展时是递增的。

#### 使用实例

##### 将`I love programming`中的所有字符`o`转换为字符`t`，字符`e`转换为字符`z`
```sh
$ echo "I love programming" | tr "oe" "tz"
I ltvz prtgramming
```
##### 将`I love programming`中的所有字符`o`和字符`e`删除
```sh
$ echo "I love programming" | tr -d "oe"  
I lv prgramming
```
##### 将`I hessssssdbbbaadffff`中的连续出现字符`s`和字符`f`变为一个
```sh
$ echo "I hessssssdbbbaadffff" | tr -s "sf"
I hesdbbbaadf
```
##### 将`I hessssssdbbbaadffff`中除了字符`I`和字符`e`和字符`f`之外的字符删除
```sh
echo "I hessssssdbbbaadffff" | tr -c -d "Ief"
Ieffff
```
### seq

&ensp;&ensp;&ensp;打印一个序列，sequence
&ensp;&ensp;&ensp;常用方法：
* seq -s 指定分隔符,`-s`后面加分隔符
* seq -w 加入前导0使得输出等宽
* seq .. FIRST
* seq .. FIRST LAST
* seq .. FIRST INCREMENT LAST



### grep

&ensp;&ensp;&ensp;查找过滤。
* -v 排除
* -B 显示匹配的行，并显示该行之前的num行
* -A 显示匹配的行，并显示该行之后的num行
* -C 显示匹配的行，并显示该行前后的num行

### tail

&ensp;&ensp;&ensp;取文件的最后几行 
* tail -n 取文件最后n行
* tail -f 监控文件的变化

### 查看图像大小

* `convert a.jpg -print "Size: %wx%h\n" /dev/null`

## 常见面试

### 取文件20行到30行内容

* `sed -n '20,30p' filename`
* `awk '{if(NR>31 && NR<19) printf $1"\n"}' filename`
* `head -30 filename | tail -11`

### 将一个目录下的所有以.sh为后缀的文件中的字符串`aaa`替换为`bbb`

* `find dirname -type f -name "*.sh" | xargs sed -i 's#aaa#bbb#g'`
* `sed -i 's#aaa#bbb#g' find dirname -type f -name "*.sh"`
* `find dirname -type f -name "*.sh" -exec sed -i 's#bbb#aaa#g' {} \;`

### dns配置文件`resolv.conf`

* `man resolv`

### `host`在企业的作用

* man host 
* 内部DNS 主机名和ip地址进行对应
* 编辑/etc/hosts

* 开发、产品、测试等人员用于通过域名测试产品
* 服务器之间的调用可以用域名（内部的DNS)，方便迁移

### 主机名

* `host 命令`
* /etc/hostname当前生效
* /etc/sysconfig/network永久生效

### fstab

* df -h查看分区情况
* 设置文件系统挂载信息文件，使得系统启动自动挂载文件系统
* /etc/fstab
* dd if=/dev/zero of=/dev/sdc bs=4096 count=100
* mount -t ext4 -o loop,noatime,noexec /dev/sdc /mnt
* mkfs.
* fsck 检查不好的处于卸载状态的磁盘
* 设备么 uuid 标签是等同的，都可以用来挂载
* fstab很重要，一旦错误将启动不了
* (1)
* (2) 救援模式 rescue
* mount -o rw,remount /

###
* cat /etc/arch-release
* uname -r 版本
* uname -m 32位还是64位
* hostname
* uname -n查看主机名和hostname一样
* uname -a所有
* useradd /etc/passwd
* /etc/shadow
* /etc/group
* passwd 
* $PS1
* whoami
* su - 用户名 `-`的作用是将当前的环境变量也切换到相应位置

### selinux

* /etc/selinux/config  /etc/systemconfig/selinux
* sed -i "s#SELINUX=DISABLED#SELINUX=ENFO#" /etc/selinux/config
* setenforce 0 临时生效
* getenforce

### 运行级别

* /etc/inittab
* runlevel 
* init切换级别
### 开机启动

* shhd
* rsyslog
* network
* crond
* sysstat
* （1）用chkconfig(2)放入/etc/rc.local服务器档案文件，放入rc.local中注释备案。
* 当挂载NFS网络文件系统时，网卡还没启动，fstab已经启动，这时fstab无法挂载网络文件系统，这时只能使用比网卡启动晚的rc.local才能完成任务。

### 打印行号的方法

* cat -n
* vi/vim :set number
* grep -n '.' filename
* grep -n '$' filename
* grep -n '^' filename
* nl filename 
* awk '{print NR,$0}' filename
* sed '=' filename 
* less -N filename

### setfacl getfacl

### 定时任务crond

&ensp;&ensp;&ensp;分两种，一种是系统自动完成的，如日志，一种是用户安装的

crontab -l crontab -e

* crond 适合周期性任务。
* at 适合执行一次，突发性任务。
* anacron 适合非7*24小时开机的任务。

crond守护进程，一直运行，crontab是管理它的命令，crontab -l 列表，crontab -e 编辑。

## 参考资料

* 各种命令的`man page`
