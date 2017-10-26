---
title: 正则表达式题目
date: 2017-09-26 21:18:16
categories:
  - 正则表达式
tag: 
  - 正则表达式
---

## 正文

&ensp;&ensp;&ensp;这里呢收集一些正则表达式的好题目进行练习。

<!--more-->

### `abba`

&ensp;&ensp;&ensp;题目：

![abba](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/abba.png?raw=true)

&ensp;&ensp;&ensp;这个题目很有意思，仔细观察发现，题目的意思就是匹配不含长度为4回文的串。如`abba`、`inflammableness`分别含有长度为4的回文`abba`和`amma`，所以不能匹配。那么怎么写呢？想想咱们小时候做数学题的时候，遇到否定的问题一般把它变为肯定再做，这里也是一样，先写出能够匹配带4-回文的串的模式，`.*(.)(.)\2\1.*`,然后呢就要加否定了，那么怎么正则表达式中怎么加否定呢？很简单，环视`(?!)`，想想当`(?!exp)`放在前面是什么作用来？排除数据！这不就是否定的功能吗？那么问题就简单了，得到模式`(?!.*(.)(.)\2\1.*).*`。而对与原题目来说，对于左边匹配只需能够匹配串的一部分即可，对于右边只需能够得到匹配失败即可。所以模式还能缩短，具体过程我就不说了，只给出最终模式`^(?!(.)+\1)|fu`。

### `Prime`

![prime](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/prime.png?raw=true)

&ensp;&ensp;&ensp;这个题目很有意思啊，仔细观察发现，其实意思就是匹配长度为素数的串，不匹配长度为合数的串，怎么做呢？还是沿用上一题的方法，先写匹配合数的，再加否定，为什么呢？因为匹配合数的好写啊，小时候的数学知识告诉我们，合数简单的来说就是有大于1小于自身的因子，也就是说，存在m大于1且小于n，使得n能整除m那么n就是合数。合数是啥搞明白了，那么怎么写正则模式呢？分组！先写m的匹配`(..+)`，这个模式代表匹配大于等于2长度的串。再写n能够整除m，n能整除m的意思也就是说很多个m合在一起可以组成n，也就是`(..+)\1+`。这里一个技巧就是`\1`分组后面也是可以加`+`和`*`的。这样最终模式也就能够写出了`^(?!(..+)\1+$).*$`。同前面一样，对于题目的特殊性，题目的最终答案可以短一些`^(?!(..+)\1+$)`