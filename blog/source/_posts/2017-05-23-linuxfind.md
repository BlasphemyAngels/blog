---
title: Linux中命令和文件的查询
date: 2017-05-23 21:58:09
categories:
  - Linux
tags:
  - linux
  - find
---


## 目录

* [whereis](#jumpwhereis)
* [which](#jumpwhich)
* [find](#jumpfind)
* [locate](#jumplocate)


## 正文

&ensp;&ensp;&ensp;&ensp;<font color=#0099ff>Linux</font>中有的时候需要检索文件和命令。这个时候`Linux`提供的强大查找命令就派上了用场。

<!--more-->

### <span id="jumpwhereis">whereis</span>

&ensp;&ensp;&ensp;搜索程序的位置。注意，它只能用于<font color=#00fff>程序名 </font>。

* whereis -b (bin) 搜索二进制文件
* whereis -m (man) 搜索man page说明文件
* whereis -s (source) 搜索源文件
* 如果省略参数则，返回所有信心

![whereis](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/whereis.png?raw=true)


### <span id="jumpwhich">which</span>

&ensp;&ensp;&ensp;作用于系统中的命令（包括用户安装的）。从PATH环境变量中指定的路径目录中查找目标命令，并返回第一个结果。所以它的执行结果就跟PATH中的路径顺序有关了，这样它就可以查看一个命令到底是执行的哪个位置的程序，比如用户装了两个 python程序，都是python2.7版本，但是安装的位置不一样（比如一个是系统自带，一个是 anaconda自带）那么怎么区分在终端敲python时执行的是哪一个呢？which就能完成区分的功能。

![which](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/which.png?raw=true)

### <span id="jumpfind">find</span>

&ensp;&ensp;&ensp;<font color=#00fff>find</font>是Linux提供的一个非常强大的一个文件查找命令。他能在给定的文件系统上查找任何你想要的文件。

命令格式：

    find [目录] [属性] [属性值] [指定动作]

具体含义：

* [目录] 要查找的目录，默认当前目录
* [属性] 要查找的文件的条件属性，换句话说，就是要根据什么来查找。如：(name, size, user, uid, 权限等)
* [属性值] 如果前面指定了查找条件属性，那么就需要在这一给定属性值，告诉命令具体根据属性的什么属性值查找文件。
* [指定动作] 在找到相应条件的文件后，对其指定的动作，如`-ls`表示找到文件后用ls 命令展示在终端

常用应用：

* find [dir] -name 
* find [dir] -group 
* find [dir] -uid
* find [dir] -type
* find [dir] -size {n(等于n) -n(小于n) +n(大于n)}
* find [dir] -size -4 -size +2  找大于2小于4的
* find -ctime 1 找创建一天的
* find -cmin 1 创建时间至少是一天的
* find -cmin +1 创建时间至少大于一天的
* find -cmin -10
* find -newer file1 找比file新的文件
* find /mnt -perm 222 找权限是222的文件
* find -perm +222 只要有一位大于等于就匹配(或关系)
* find -perm -222 每一位都需要有w即可，(和关系)
* find -perm -2(o)
* find -perm -22(g-o)
* find -perm -222(u-g-o)

### <span id="jumplocate">locate</span>

locate 从/var/lib/mlocate/mlocate.db中查找要查找的文件，这个数据库默认每天更新一次，可以手动用命令:`upatedb`命令更新数据库。它的功能和`find -name`类似，但是它的效率高，因为`locate`查找数据库，而`find`是检索磁盘文件系统。

* locate -i(ignore case)

## 参考链接
* [Linux下的五个查找命令：grep、find、locate、whereis、 which](http://www.cnblogs.com/wanqieddy/archive/2011/07/15/2107071.html)
