---
layout: post
title: "Linux中的文件和文件系统的压缩与打包"
date: 2017-03-11 
categories:
  - Linux
tags: 
  - linux
  - 压缩
---


## 正文

&nbsp;&nbsp;　&nbsp;Linux提供了一些文件的压缩和打包命令，现在就让我们一起来学习他们。其中每个命令有很多参数，这里只是说明一些常用的，其它一些可以通过man page来查看。

<!--more-->

### Linux常用的压缩命令

#### compress 命令

&nbsp;&nbsp;　&nbsp;compress命令使用“Lempress-Ziv”编码压缩数据文件。compress是个历史悠久的压缩程序，文件经它压缩后，其名称后面会多出".Z"的扩展名。当要解压缩时，可执行uncompress指令。事实上uncompress是指向compress的符号连接，因此不论是压缩或解压缩，都可通过compress指令单独完成。这个命令已经过时了，已经不常用，这里为了完整性，也解释一下。


* -f：不提示用户，强制覆盖掉目标文件；
* -c：将结果送到标准输出，无文件被改变；
* -r：递归的操作方式；
* -b\<压缩效率\>：压缩效率是一个介于9~16的数值，预设值为"16"，指定愈大的数值，压缩效率就愈高；
* -d：对文件进行解压缩而非压缩；
* -v：显示指令执行过程；
* -V：显示指令版本及程序预设值。

#### uncompress命令

&nbsp;&nbsp;　&nbsp;uncompress命令用来解压缩由compress命令压缩后产生的“.Z”压缩包。

* -f：不提示用户，强制覆盖掉目标文件；
* -c：将结果送到标准输出，无文件被改变；
* -r：递归的操作方式。

一般用法:<br/>
`compress -v filename`<br/>
`compress -c -v filename > targetfilename`

#### gzip命令

&nbsp;&nbsp;　&nbsp;gzip, zcat是Linux提供的一对操作文件压缩的命令。gzip对文件进行压缩解压，zcat查看gzip压缩的文件。

*  `gzip filename` 压缩
*  `zcat` 查看gzip压缩完的内容
*  `gzip -d filename` 解压
*  `gzip -c filename > filename` 保留源文件

#### bzip2, bzcat命令

&nbsp;&nbsp;　&nbsp;bzip2和gzip一样的用法。

#### zip用法

* `zip aaa.zip aaa`  压缩aaa文件
* `unzip aaa.zip`  解压

#### rar命令

&nbsp;&nbsp;　&nbsp;另外有的时候需要rar，unrar命令，如果系统不提供，可以自行下载。

##### unrar命令
* unrar e Extract files without archived paths
* unrar x Extract files with full path

### Linux的归档命令 tar

&nbsp;&nbsp;　&nbsp;很多时候，比如你要传输文件时，有许多文件需要传输，由于文件数很多，所以依次传输的话太繁琐，这个时候如果能够把这些文件组合成一个整体一次传输，传输完成后再分解成多个文件就方便了。幸运的是，Linux提供了这个命令，即tar命令。

#### 选项

* -f 指定要归档文件的名字
* -c 对文件归档
* -v 显示归档过程
* -x 对文件解档
* -t 查看归档文件内容

#### 常用命令

* tar -cvf aaa.tar aaa 将文件夹aaa下的文件归档，归档后名字为aaa.tar
* tar -cvf aaa.tar aaa --remove-files 归档文件并删除源文件
* tar -tvf hosts.tar 在不解档的情况下查看文件内容
* tar -xvf aaa.tar 解档
* tar -xvf aaa.tar -C aa/ 解档到aa目录下
* tar -xvf aaa.tar a1 只解档aaa.tar中的a1文件

##### tar可以跟gzip或bzip2组合使用

* tar -zcvf xxx.tar.gz xxx 归档并压缩
* tar -zxvf xxx.tar.gz 解压并解档
* tar -jcvf xx.tar.bz2 xx 归档并压缩
* tar -jxvf xx.tar.bz2b 解压并解当

### Linux的备份命令

#### 备份的种类

* 完全备份
&nbsp;&nbsp;　&nbsp;每次都将原文件系统备份，耗费空间。
* 差异备份
&nbsp;&nbsp;　&nbsp;每次在原文件系统的基础上做新数据的备份，容易还原。
* 增量备份
&nbsp;&nbsp;　&nbsp;每次在前一次备份的基础上做新数据的备份，还原麻烦，但是节省空间。


#### dump 命令

&nbsp;&nbsp;　&nbsp;dump是linux提供的备份工具程序，可将目录或整个文件系统备份至指定的设备，或备份成一个大文件。这里重点注意，dump针对的文件系统而不是单个文件。

##### 语法

* `dump [-cnu][-0123456789][-b <区块大小>][-B <区块数目>][-d <密度>][-f <设备名称>][-h <层级>][-s <磁带长度>][-T <日期>][目录或文件系统]`
* ` dump [-wW]`

##### 选项

* -0123456789 　备份的层级。
* -b<区块大小> 　指定区块的大小，单位为KB。
* -B<区块数目> 　指定备份卷册的区块数目。
* -c 　修改备份磁带预设的密度与容量。
* -d<密度> 　设置磁带的密度。单位为BPI。
* -f<设备名称> 　指定备份设备。
* -h<层级> 　当备份层级等于或大雨指定的层级时，将不备份用户标示为"nodump"的文件。
* -n 　当备份工作需要管理员介入时，向所有"operator"群组中的使用者发出通知。
* -s<磁带长度> 　备份磁带的长度，单位为英尺。
* -T<日期> 　指定开始备份的时间与日期。
* -u 　备份完毕后，在/etc/dumpdates中记录备份的文件系统，层级，日期与时间等。
* -w 　与-W类似，但仅显示需要备份的文件。
* -W 　显示需要备份的文件及其最后一次备份的层级，时间与日期。

##### 常用命令

* dump -0u -f filesystem dumpname
* restore -t xxx.dump #cat  the content of the dump file
* restore -r(restore) -f 恢复文件
* restore -i -f aaa.dump 进入交互模式

##### dump的重要特性

&nbsp;&nbsp;　&nbsp;dump -0 会完成完整的系统备份，以后的备份，dump会做对前面的备份等级中小于当前等级的最大的备份的增量备份。也就是说，比如，前面已经做了0,2,3,4,6级备份，当前要做5级备份，由于0,2,3,4,6中小于5级的最大等级是4级，所以当前五级备份做的是对前面四级备份的增量备份。

&nbsp;&nbsp;　&nbsp;通过dump这样的特性，可以完成对文件系统的增量备份（按0,1,2...按递增等级顺序备份),也可以完成差异备份(按n,n-1,n-2,...的递减等级顺序备份)。


##### 跳过文件

&nbsp;&nbsp;　&nbsp;如果想要让文件系统中的某个文件不备份，可以使用chattr命令。<br/>

`chattr +d <filename>`向文件添加一个属性，使其在dump备份时跳过此文件。


#### 光盘写入工具 mkisofs

`mkisofs -o xx.iso file1 file2 ...`<br/>
`cp /mnt/cdrom xx.iso # copy the CDrom to iso`<br/>
`mount -o loop`


#### dd命令

&nbsp;&nbsp;　&nbsp;dd 是 Linux/UNIX 下的一个非常有用的命令，作用是用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。

##### 选项

* if =输入文件（或设备名称）。
* of =输出文件（或设备名称）。
* ibs = bytes 一次读取bytes字节，即读入缓冲区的字节数。
* skip = blocks 跳过读入缓冲区开头的ibs*blocks块。
* obs = bytes 一次写入bytes字节，即写入缓冲区的字节数。
* bs = bytes 同时设置读/写缓冲区的字节数（等于设置ibs和obs）。
* cbs = byte 一次转换bytes字节。
* count=blocks 只拷贝输入的blocks块。
* conv = ASCII 把EBCDIC码转换为ASCIl码。
* conv = ebcdic 把ASCIl码转换为EBCDIC码。
* conv = ibm 把ASCIl码转换为alternate EBCDIC码。
* conv = block 把变动位转换成固定字符。
* conv = ublock 把固定位转换成变动位。
* conv = ucase 把字母由小写转换为大写。
* conv = lcase 把字母由大写转换为小写。
* conv = notrunc 不截短输出文件。
* conv = swab 交换每一对输入字节。
* conv = noerror 出错时不停止处理。
* conv = sync 把每个输入记录的大小都调到ibs的大小（用NUL填充）。
* dd if=[STDIN] of=[STDOUT]  输入和输出
* seek=BLOCKS 跳过一段以后才输出

##### 拷贝光盘

&nbsp;&nbsp;　&nbsp;注意：光碟是标准的 iso9660格式才行

`dd if=/dev/cdrom of=cdrom.iso`<br/>
`cdrecord -v cdrom.iso`<br/>

##### 特殊块

* dd if=/dev/zero of=hello.txt bs=20M count=1 创建一个20M的空文件
* /dev/null，外号叫无底洞，你可以向它输出任何数据，它不会达到饱和。
* /dev/zero,是一个输入设备，你可你用它来初始化文件。
* /dev/null------它是空设备，也称为位桶（bit bucket）。任何写入它的输出都会被抛弃。如果不想让消息以标准输出显示或写入文件，那么可以将消息重定向到位桶。
* /dev/zero------该设备无穷尽地提供0，可以使用任何你需要的数目——设备提供的要多的多。他可以用于向设备或文件写入字符串0。

#### cpio + find 备份

* find -name file \| cpio -o > xx.cpio
* cpio -i \< xx.cpio

## 参考链接:    
[Linux-dd命令详解](http://www.cnblogs.com/dkblog/archive/2009/09/18/1980715.html)

<br>

转载请注明:[Artemis的博客]([https://BlasphemyAngels.github.io)--> [点此看原文
](https://blasphemyangels.github.io/2017/03/Linuxcompress/)
