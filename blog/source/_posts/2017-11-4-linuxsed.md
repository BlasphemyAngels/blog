---
title: Linux三剑客之sed
copyright: true
date: 2017-11-03 22:02:18
password: 1314
mathjax:
categories:
  - Linux
tags:
  - linux
  - sed
---

## 

### sed

&ensp;&ensp;&ensp;sed(stream editor)流编辑器

* sed -n 取消自动打印（sed会自动将全部未过滤内容输出）
* sed -i 将更改作用到源文件，不加`-i`则只会把`sed`的输出修改而不会改变源文件

&ensp;&ensp;&ensp;sed替换用法：

* `sed 's#string1#string2#g' filename`
&ensp;&ensp;&ensp;其中`s`代表`sub`，意思是替换，`#`是分隔符，`string1`要替换的文本，`string1`要替换成的文件，`g`是`global`全局的意思，可以不带`g`，不带`g`的意思是值替换第一个（想想`vim`中的替换）。
&ensp;&ensp;&ensp;`#`可以换成别的字符，但是要换的话，3个`#`都得换，换成的字符不能是`s`、`g`（当`s`或`g`出现在命令体中时）或`string1`和`string2`中的任何字符，如果非得使用上述字符，则加反斜线`\\`。
sed 流编辑器 增删改查 过滤 取行
<!--more-->

sed [options] [sed-command] [input]

`sed命令`前面可以加地址范围`n1[,n2]`
* `10[sed-command]`
* `10,20[sed-command]`
* `10,+20[sed-command]`
* `1～2[sed-command]`
* `10，$[sed-command]`

地址还支持正则匹配
* `/libai/[sed-command]`
* `/libai/,/luna/[sed-command]`
* `/libai/,$[sed-command]`
如果匹配有多个，回想`sed`的流程来分析

地址还支持数字和正则匹配混合
* `10,/libai/[sed-command]`
* `/libai/,10[sed-command]`
* `/libai/,+10[sed-command]`

### 增删改查

* `a` 追加 `sed '2a hehehe' filename`
* `i` 插入 `sed '2i hehehe' filename`
* `a`和`i`的区别可以想想`vim`的`a`和`i`
* `a`和`i`可以单行增加也可以多行增加
* `sed 2a hehehe\nhohoho filename`
* 多行时可以在文本中显示加入`\n`也可以在写命令要加入文本时在终端显式敲回车
* `d` 删除 `sed 'd' filename`全部删除
* `sed '2d' filename`
* `sed '2,5d' filename`
* `c` 行间替换,用法跟前面的`a`和`i`、`d`相似
* `s` 替换指定字符串，`g`命令替换标志---全局替换标志
* `sed 's#str1#str2#g' filename`将str1替换为str2
* `-i`参数 不加此参数，那么sed所做的任何操作都是在模式空间中做的，也就是在文件在内存中的映像做的，原文件内容是不变的，加了这个参数，就将在模式空间做的更改更新到文件上。
* 变量替换 `sed s#$x#$y#g zimux.txt` 将zimux.txt中变量x中的值替换为变量为y的值。这种情况下，sed-command不能使用单引号，可以使用双引号和什么都不加。非要用单引号，可以`sed 's#'$x'#'$y'#' filename`也可以使用`eval sed 's#$x#y#g' filename`。
* 分组替换 跟正则表达式的分组替换和vim中的分组匹配替换是一样的。`echo 'abc123' | sed 's#[a-z]+([0-9]+)#\1#g'`
* `-r`选项启动扩展正则表达式
* 特殊符号& 代表被匹配的内容,相当于`\0`第0组
企业案例：批量重命名文件
`ls *.jpg | sed -r 's#(.*)_finished.*#mv & \1.jpg#g'`
* `rename`命令
* `p`打印输出的内容，常与`-n`选项配合使用，`-n`选项取消默认输出。其实默认情况下会对匹配到的内容输出了，`p`又输出了一次。所以加上`-n`取消默认输出。
* set 修改文件及另存文件及替换命令
* 修改文件 `-i`
* 备份 `sed -i[SUFFIX] 'set-commant' filename`
* 另存文件 `sed 'w outputfile' filename` 将模式空间的内容另存到`outputfile`文件中
* `sed '[地址范围][模式范围] s#[替换的字符串]#[替换后的字符串]#[替换标志]' [输入文件]`
* 替换标志  全局标志g  数字标志1，,2，,3，...  打印p 写w 忽略大小写i  执行命令标志e
* 地址范围是指定哪一行放入模式空间操作。
* 全局标志g 不带g代表已经被前面选中的东西中匹配的第1列 带g表示匹配的所有列
* `Ms# # #Ng`
* Ms 对第M行进行操作 不带g表示对匹配的第一列进行操作 带g 表示匹配的所有列
* Ng 从第N处/列开始匹配
* Ms Ng 从第M行的第N出匹配进行处理
* 数字表示X 只对第X处/列替换
* 打印`p`
* `sed 's#ab#dd#;w outputfile' filename` 写命令 `w`
* `sed 's#ab#dd#w outputfile' filename` 写标志 `w`
* 两种是不同的
* 忽略大小写标志 `i`
* 执行标志e 将模式空间的任何内容当做shell bash 命令执行
* 案例 系统开机启动优化
* `sed -r 's#(.*),(.*),(.*)#\L\3,\E\1,\U\2#g' filename`
* 特殊符号= 获取行号
* 一条sed 命令执行多行sed命令 删除第三行到末尾的数字 并将10替换为01
* `sed -e '3,$d' -e 's#10#01#g' filename`
* `sed '3,$d;s#10#01#g' filename`
* `sed -f person.sed filename` `person.sed`是一个sed脚本
* 案例 一个文件100行，把第5，35，,70行单独拿出来
* `sed -n '3p;35p;70p' filename`
* 特殊符号{}的用法
* `sed -n '2,4{p;=}' filename`
* l命令 打印中包含不可见字符
* 转换字符 `tf 'abc' 'ABC' < filename`
* sed 'y#abc#ABC#' filename
* 退出sed命令q 执行到某个地方不再执行
* 从文件中读取数据 命令r 
* sed 'r num.txt' filename
* 保持空间和模式空间
* 模式空间：
* n 清空当前模式空间，并读入下一行
* N 不清空当前模式空间，并读入下一行，并用\n连接模式空间的两行
* 案例 用户名密码文件变为 stu=aaa 
* `sed 'N;s#\n#=#' filename`
* sed操作多个文件
* 模拟其他命令
* cat sed -n 'p'  sed 'N' sed 'n' sed 's# # #'
* grep sed -n '//p' sed -n '// !p' 
* head sed -n '1,3p' sed '2q'
* wc sed -n '$='
