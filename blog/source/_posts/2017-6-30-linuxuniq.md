---
title: linux中的uniq命令
copyright: true
date: 2017-06-30 23:54:11
categories:
  - Linux
tags:
  - linux
  - 命令
---

<blockquote>
uniq是lunix中一个很常用的命令，常用来做文本处理，对文本行进行去重过滤。
</blockquote>

#### 命令讲解

找出或忽略重复的行。

命令格式:
```sh
uniq [OPTION]... [INPUT [OUTPUT]]
```

对输入的数据中相邻的行进行过滤输出到给定的输出项上。默认情况下是将相邻行中重复的项去掉，只留下重复项中的第一个。

这里要注意，`uniq`是对`相邻行`进行过滤，所以要使`uniq`达到预期结果，一般要先对`uniq`的输入进行`排序`。

常用选项：

* `-c, --count`

    过滤掉重复行，并在输出的每一行前面加上此行出现的次数。
* `-d, --repeated`
    
    只显示在文件中重复的行。
* `-u, --unique`
    
    只显示文件中不重复的行。

* `-n` 

    在进行每一行的比较时，前`n`个字段以及他们之间的空白都不参与比较。
    
* `+n`

    在进行每一行的比较时，前`n`个字符不参与比较。
* `-f n`
    
    与`-n`相同，这里`n`是字段数。
* `-s n`
    
    与`+n`相同，这里`n`是字符数。

#### 实验结果

```sh
# 文件内容
$ cat test
a hehe a
b hehe b
c hehe c
c hehe d
c hehe d
f hehe f
f hehe f

```
注意在文件中如果只有一个回车，那么此行会被认识空行也会被任务是一行，输出。

```sh
# 过滤行并显示次数。
$ uniq -c test
1 a hehe a
1 b hehe b
1 c hehe c
2 c hehe d
2 f hehe f
1
```

```sh
# 只显示重复的行。
$ uniq -d test
c hehe d
f hehe f
```

```sh
# 只显示没有重复的行。
$ uniq -u test
a hehe a
b hehe b
c hehe c
```

```sh
# 忽略第一个字段。
$ uniq -f 1 test
a hehe a
b hehe b
c hehe c
c hehe d
f hehe f
```

```sh
# 忽略第一个字符，其实效果和上一个一样。
$ uniq -s 1 test
a hehe a
b hehe b
c hehe c
c hehe d
f hehe f
```

### 参考内容

* `Linux man page`


