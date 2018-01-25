---
title: linux中的sort命令
copyright: true
date: 2017-08-01 11:10:33
categories:
  - Linux
tags:
  - linux
  - sort
  - 命令
---

<blockquote>
sort是linux中用于文本行排序的命令，它常和uniq命令搭配使用，是在linux文本处理中常用到的命令。
</blockquote>

#### 命令讲解

命令格式:
```sh
sort [选项]...[文件]...
```
将所有文件的行进行排序输出。如一个文件也没有指定或者指定的文是`-`，那么将从标准输入读入内容。

常用选项：
* `-c,--check` 
    
    对检查文件中的行是否已经排好序，如果文件乱序，则输出第一个乱序的行的相关信息，最后返回1。这个选项本身不会对数据排序。

* `-r,--reverse`
    
    `sort`默认升序排序，`-r`指定为降序排序。

* `-u,--unique`
        
    对结果进行去重，如果一行内容出现多次，只保留第一次。
* `-n,--numeric-sort`
    
    只对出现的数字排序,其他非数字的内容忽略，如一行的内容是`ab2def3sd4`，那么在排序时只会以`234`作为一行的内容进比较排序。
* `-t,--field-separator=SEP`
    
    指定分隔符`SEP`,使用`SEP`作为一行中字段的分隔符，而不是使用`空白`。

* `-k,--key=KEYDEF`
    
    `sort`默认是使用整行进行排序，如果指定了`-k`选项，那将不使用整行而是使用行的一部分内容作为键进行排。
    
    `KEYDEF=pos1[,pos2]`

    表示根据第`pos1`字段到`pos2`字段之间的内容进行排序，如`-k 2`就是根据第二个字段进行排序。
    
    每个`pos`还能指定从第几个字符开始进行排序。如`-k3.1,4.3`表示根据从第三个字段的第`1`个字符到第四个字段的第`3`个字符之间的内容进行排序。
    
    一条`sort`命令可以指定多个`-k`选项，如`-k 3 -k 4`表示先根据第三字段进行排序，如果第三字段相同则根据第四字段排序。
    
    在`-k`中还可以指定其他选项，如`-k 3nr`表示根据第三字段中的数字进行反向排序。
    
    现在就能看懂`KEYDEF`的真正定义了：<br/>
    `[ FStart [ .CStart ] ] [ Modifier ] [ , [ FEnd [ .CEnd ] ][ Modifier ] ]
`

#### 实验

```sh
# 查看文件内容
$ cat test
c2bsf2-c2-hehe
e3gsf5-c1-hehe
d1esf1-c5-hehe
b0dsf3-c4-hehe
```

```sh
# 排序
$ sort test
b0dsf3-c4-hehe
c2bsf2-c2-hehe
d1esf1-c5-hehe
e3gsf5-c1-hehe
```

```sh
# 检查文件是否排好序
$ sort -c test
sort: te:3: disorder: d1esf1-c5-hehe
```

```sh
# 降序排列
$ sort -r test
e3gsf5-c1-hehe
d1esf1-c5-hehe
c2bsf2-c2-hehe
b0dsf3-c4-hehe
```

```sh
# 根据数字进行排序
$ sort -n test
b0dsf3-c4-hehe
c2bsf2-c2-hehe
d1esf1-c5-hehe
e3gsf5-c1-hehe
```

```sh
# 去重
$ cat file
f
e
d
f
d
e
$ sort -u file
d
e
f
```

```sh
# 指定分隔符并根据第一个字段进行排序
$ sort -t '-' -k 2 test
e3gsf5-c1-hehe
c2bsf2-c2-hehe
b0dsf3-c4-hehe
d1esf1-c5-hehe
```

```sh
# 指定分隔符并根据从第一个字段的第二个字符到第二个字段的第一个字符之间的数字内容进行降序排序
$ sort -t '-' -k1.2,2.1nr test
e3gsf5-c1-hehe
c2bsf2-c2-hehe
d1esf1-c5-hehe
b0dsf3-c4-hehe
```

### 参考内容
* `Linux man page`
* ![linux之sort用法](https://www.cnblogs.com/dong008259/archive/2011/12/08/2281214.html)
