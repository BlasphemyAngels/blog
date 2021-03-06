---
title: 双拆分数
copyright: true
date: 2018-01-09 22:50:42
password:
mathjax:
categories:
  - 算法
  - 面试
tags:
  - 双拆分数
  - algorithm
---

#### 题目描述

对于一个数字串 s，若能找到一种将其分成左右两个非空部分 s1,s2 的方案，使得：
* s1,s2 均无前导零
* 存在两个正整数 a,b，使得 b 整除 a，且 a/b=s1, a*b=s2

那么我们记这是一个合法的分法。特别地，如果一个串有两个或更多个不同的合法的分法，那么我们称这个数字串是双拆分数字串。

给定一个 n，要求构造一个长度恰为 n 的双拆分数字串。如果无解，输出 -1。

<!--more-->

#### 思路

首先根据上面的条件可以得出，一种合法的分法也就等价于`s2/s1=k*k`，也就`s2/s1`的结果是完全平方数。

然后就是找到一个长度为`n`的数，它有两种合法的分法。

当`n<=3`时，显然无解，因为只有两个分割点，而且第二个分割点的分割使得`s2<s1`。

我们又会想到，如果一个数`n`是双拆分数，那么在`n`后面加两个`0`得到的数肯定也是双拆分数。那么问题就简单了，因为`n<=3`的双拆分数不存在，我们只需找到一个长为`4`和一个长为`5`的双拆分数即可，而其他任意长度的双拆分数我们都可以使用这两个双拆分数在后面添`0`得到。

可以写一个暴力程序得到一个长度为`4`的双拆分数为`1144`，长度为`5`的双拆分数为`16400`。当然对数学敏感的人应该很快就能构造出这两个数。
