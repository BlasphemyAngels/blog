---
layout: post
title: "Linux之权限管理"
date: 2017-03-16 
categories:
  - Linux
tags: 
  - linux
  - 权限
---

<!--<img src="http://chart.googleapis.com/chart?cht=tx&chl= P(A)=\sum_{i=1}^{n}P(AB_i)" style="border:none;">-->

## 正文

### ll命令

ll命令是linux中列出文件（夹）详细信息的命令，相当于ls -l

![auth1](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/authority.png?raw=true)

如图所示，观察文件a前面的详细信息，可以分为几段：

#### 第一段

一共十位，第一位表示文件的类型，`-`代表普通文件，`d`代表目录文件，`b`代表块设备文件，`c`代表字符设备文件，`s`代表socket文件，`p`代表管道文件，`l`代表软连接文件。

其余的九位每三位分成一组，共分成三组，然后每组分别代表用户(user)，用户所属组 (group)，不属于用户组的其他用户(others)。然后每组中三位分别表示读权限，写权限和执行权限。

权限和命令间的关系：
* r 对于文件代表是否能用cat等命令进行查看。对于目录，代表能否用ls等命令查看目录内的内容。
* w 对于文件代表能否改变文件内的内容。对于目录代表能否改变目录内的内容（添加、删除文件等）。
* x 对于文件代表文件(是否能运行)，对于文件夹代表是否能cd登录进去。

<!--more-->

#### 第二段

是指文件的个数，如果是文件那么此段值就是１，如果是文件夹则显示其中含有文件的个数。

#### 第三段

是指文件所属的所有者

#### 第四段

是指文件所有者所属的组

#### 第五段

文件的大小，默认以byte为单位。空目录一般显示1024byte。

#### 第六段

表示文件最后被修改的时间

#### 第七段

文件名

### 权限

Linux的权限主要就是上面ll中列出的第一段的后九位，即用户，用户组，其他用户对文件的读，写和执行权限。下面就介绍Linux中对权限的操作命令。

### umask命令

接着前面说，上面说的９位表示权限的位，每一位的取值就两种可能，一种是具备这种权限，一种是不具备。这样的话如果用１表示具备，０表示不具备，那么这九位表示权限的位就能转化位九位数字来表示。如rwxr-x---就能用111101000来表示。再将这九位数字每三位分一组，每组数就可以看做一个二进制数。111101000->(111)(101)(000)。然后再将每组数转换位十进制(111)(101)(000)->750。这样权限就能用三位十进制数表示了，但是每一位的值不大于７。

接着说`umask权限过滤符`。在Linux中每次创建新文件时，都会用umask的值对文件的权限进行过滤，从而得到文件初始权限。比如umask的值是022，按照上面的讨论可以反向的将022 转换为９位的权限值022->000010010->----w--w-，然后新文件时用----w--w-对 rwxrwxrwx进行过滤(相同的过滤掉)后得到的权限就是文件的初始权限。这里得到的是 rwxr-xr-x，这就是umask的作用。在命令行之间敲umask可以查看umask的值，`umask value`命令可以设置umask值。 `umask -S`命令会显示带符号的权限信息，而不是数字。比如umask的值为022，那么 `umask -S`会显示`u=rwx,g=rx,o=rx`。

### chmod命令

chmod命令用于给文件赋权限。用法很简单，如果想给文件增加所有者的写权限，则用 `chmod u+w filename`,同样的chmod后面的权限操作符由三部分构成，第一部分是指出要修改权限的对象，如u、g、o，第二部分是指出要做的操作，增加权限用+，减少权限-，第三部分指出要增加或减少的权限rw或x。如要为文件a增加其他用户的写权限，命令为 `chmod o+w a`。chmod也可以直接向文件赋予上述的三位数字的权限，如要向文件赋予 `rwxr-xr-x`权限，则命令为`chmod 755 filename`。

如：

* `chmod 4644 = chmod u+s`
* `chmod 2644 = chmod g+s`
* `chmod 1644 = chmod o+t`

### chattr命令

改变文件的属性。可以用`lsattr`命令查看文件的属性。与chmod这个命令相比，chmod只是改变文件的读写、执行权限，更底层的属性控制是由chattr来改变的。

* \+ ：在原有参数设定基础上，追加参数。
* \- ：在原有参数设定基础上，移除参数。
* = ：更新为指定参数设定。
* A：文件或目录的 atime (access time)不可被修改(modified), 可以有效预防例如手提电脑磁盘I/O错误的发生。
* S：硬盘I/O同步选项，功能类似sync。
* a：即append，设定该参数后，只能向文件中添加数据，而不能删除，多用于服务器日志文件安全，只有root才能设定这个属性。
* c：即compresse，设定文件是否经压缩后再存储。读取时需要经过自动解压操作。
* d：即no dump，设定文件不能成为dump程序的备份目标。
* i：设定文件不能被删除、改名、设定链接关系，同时不能写入或新增内容。i参数对于文件 系统的安全设置有很大帮助。
* j：即journal，设定此参数使得当通过mount参数：data=ordered 或者 data=writeback 挂 载的文件系统，文件在写入时会先被记录(在journal中)。如果filesystem被设定参数为 data=journal，则该参数自动失效。
* s：保密性地删除文件或目录，即硬盘空间被全部收回。
* u：与s相反，当设定为u时，数据内容其实还存在磁盘中，可以用于undeletion。

各参数选项中常用到的是a和i。a选项强制只可添加不可删除，多用于日志系统的安全设定。而i是更为严格的安全设定，只有superuser (root) 或具有CAP_LINUX_IMMUTABLE处理能力（标识）的进程能够施加该选项。

### SUID

set user ID 

`chmod u+s` 如果权限中带着S位那么所有用户都能像当前这个用户一样使用这个被赋予权限的命令或文件。

使用SUID时要注意几点：

* SUID只对二进制文件有效
* 调用者对该文件有执行权
* 在执行过程中，调用者会暂时获得该文件的所有者权限
* 该权限只在程序执行的过程中有效

SUID的执行过程(图片转自下面参考链接的博客):

![suid](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/suid.png?raw=true)

### SGID

Set GID 当SGID修饰文件时，它的作用和SUID相似，用户将得到该文件所属组的权限。当
SGID修饰目录时，而且此时用户对目录有写权限，那么用户在目录中创建的任何东西都继
承到目录所属的组。

`chmod g+s [name]`

### SBIT

Sticky Bit

只能用于文件夹，在others权限的执行位上加上一个t位。当某一个目录拥有SBIT权限时，则任何一个能够在这个目录下建立文件的用户，该用户在这个目录下所建立的文件，只有该用户自己和root可以删除，其他用户均不可以。

`chmod o+t [name]`

也可以使用数字为文件系统增加SUID、SGIT、SBIT权限。

* 4表示SUID
* 2表示SGID
* 1表示SBIT

比如`chmod 4777 [name]`就给文件增加了SUID，如果想同时为文件增加多种权限则将相应值相加即可，比如`chmod 6777 [name]`就为文件同时增加了SUID和SGID。


## 参考链接:<br/>
[linux下ll命令列出的文件类型
](http://1744193.blog.51cto.com/1734193/490277)<br/>
[（总结）Linux的chattr与lsattr命令详解](http://www.ha97.com/5172.html)<br/>
[linux中SUID，SGID和SBIT的奇妙用途
](http://blog.csdn.net/xiaocainiaoshangxiao/article/details/17378611)

<br>

转载请注明:[Artemis的博客]([https://BlasphemyAngels.github.io)--> [点此看原文
](https://blasphemyangels.github.io/2017/03/authority/)
