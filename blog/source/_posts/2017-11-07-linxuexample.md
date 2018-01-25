---
title: linux经典案例
copyright: true
date: 2017-11-07 08:57:20
password: 1314
mathjax:
categories:
  - Linux
tags:
  - linux
  - example
---


<blockquote>整理一下遇到的linux命令的题目，并对答案做个总结</blockquote>

<!--more-->

### 统计单词数目并排序

&ensp;&ensp;&ensp;统计一个文件中每个单词各自出现的次数并排序


* sed -r 's# +#\n#g' words.txt | sed  -r '/^$/d' | sort | uniq -c | sort -r | awk '{print $2,$1}'
* cat words.txt | tr -s ' ' '\n' | sort | uniq -c | sort -r | awk '{ print $2, $1 }'
* tr -s ' ' '\n' < words.txt|sort|uniq -c|sort -nr|awk '{print $2, $1}'
* cat words.txt | awk '{for(i=1;i<=NF;++i){count[$i]++}} END{for(i in count) {print i,count[i]}}' | sort -k2nr
* awk '{for(i=1;i<=NF;i++) a[$i]++} END {for(k in a) print k,a[k]}' words.txt | sort -k2 -nr

### 打印文件第十行

&ensp;&ensp;&ensp;打印一个文件的第十行内容

&ensp;&ensp;&ensp;做法有很多：
* head -n 10 file.txt | tail -n +10
* sed -n '10p' file.txt
* awk 'NR == 10' file.txt 
* awk '{if(NR==10) print $0}' file.txt

### 找出不合法电话号码

&ensp;&ensp;&ensp;给你一个文件，文件中每行是一个电话号码，有合法的有不合法的，找出合法的电话，合法的电话号码形式是：(xxx) xxx-xxxx或xxx-xxx-xxxx，每一行不含前导或后导空格。

&ensp;&ensp;&ensp;如文件内容如下:

987-123-4567
123 456 7890
(123) 456-7890

&ensp;&ensp;&ensp;你需要输出：

987-123-4567
(123) 456-7890

* grep -P '^(\d{3}-|\(\d{3}\) )\d{3}-\d{4}$' file.txt
* sed -n -r '/^([0-9]{3}-|\([0-9]{3}\) )[0-9]{3}-[0-9]{4}$/p' file.txt
* awk '/^([0-9]{3}-|\([0-9]{3}\) )[0-9]{3}-[0-9]{4}$/' file.txt

### 转置文件内容

&ensp;&ensp;&ensp;如下所示，文件内容：
name age
alice 21
ryan 30
Output the following:
&ensp;&ensp;&ensp;需要得到：
name alice ryan
age 21 30

* awk 'NF!=0 {for(c=1;c<=NF;c++) mtx[NR,c]=$c; rows++; cols=NF;} END{for(c=1;c<=cols;c++) { line=mtx[1,c]; for(r=2;r<=rows;r++) { line=line" "mtx[r,c]}; print line; }}' file.txt


### 分析日志找访问量最大文件

分析图片服务日志，把日志（每个图片访问次数×图片大小的总和）排行，取top10，也就是计算每个url的总访问量大小。

* awk '{print $7"\t" $10}' filename | sort |uniq -c|awk '{print $1*$3,$1,$2}' | sort -rn | head 
* awk '{S[$7]++;T[$7]+=$10}END{for(k in S) print k,S[k],T[k]r' filename
