---
title: vim宏录制
date: 2017-06-03 17:16:48
categories:
  - vim
tags:
  - vim
  - 宏录制
---

## 正文

&ensp;&ensp;&ensp;在vim编辑文件时，有时会遇到要执行同一个动作序列多次的情况，这时就会用到宏录制。

下面我们就从例子出发讲解宏录制。

<!--more-->

比如文本中的内容是：

```python
this is a dog
hehehehe
woaini
you is a dog
fuck you
ho ho ho
I love you
```

现在问题来了，我想在文本的每一行前面加入一个`#`号。怎么办？先考虑如果是一行的话怎么改？ 

先把光标停在行的第一个字符，然后按`i`键进入插入模式，然后输入`#`，再按`Esc`键退出插入模式，最后按`j`键到达下一行行首。这样就改完了一行，如果改下一行可以在下一行行首按`.`键，但是就这样一行一行的改？那么如果有一百行咋办？那么这就需要宏录制了，宏录制的原理就是将一系列的操作记录下来，然后如果有要进行这系列操作多次的话，只需要播放这个记录多次即可。就类似复读机。 

### 宏录制步骤

* 按下`q`间和一个字母键，如`qa`、`qb`、...
* 进行一系列操作
* 按下`q`键，这样前面进行的一系列操作就保存在名为前面按下的字母键的记录中
* 按下`number@`加前面按下的字母键即可让前面的一系列操作执行多次

### 实验案例

还是以前面的案例为准

```python
this is a dog
hehehehe
woaini
you is a dog
fuck you
ho ho ho
I love you
```

* 将光标移到第一行的地一个字符上
* 按下`qa`键
* 然后按`i`键进入插入模式，然后输入`#`，再按`Esc`键退出插入模式，最后按`j`键到达下一行行首
* 按下`q`键
* 按下`6@a`键（为什么是6?一共7行，已经修改了1行，还剩6行）
* 完成操作

实验结果：
```python
#this is a dog
#hehehehe
#woaini
#you is a dog
#fuck you
#ho ho ho
#I love you
```

转载请注明:[Artemis的博客]([https://blasphemyangels.github.io/)--> [点此看原文
](https://blasphemyangels.github.io/2017/06/03/2017-06-03-vimdefine/)
