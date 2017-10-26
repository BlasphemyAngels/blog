---
title: Markdown中使用公式
date: 2017-10-13 11:13:23
categories:
  - 工具
tags:
  - tools
  - markdown
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
## 正文

&ensp;&ensp;&ensp;markdown作为一种简洁方便的写作方式，广受博主们的喜爱，但是有的时候需要向markdown中加入公式，该怎么办呢？
<!--more-->
&ensp;&ensp;&ensp;我一般使用两种方式，一种是使用在线公式编辑器编辑公式保存为图片，然后将图片上传到github等平台，然后在markdown中关联即可。

&ensp;&ensp;&ensp;另一种是使用`MathJax`引擎。

&ensp;&ensp;&ensp;使用方式很简单：

* 将下面代码加入到markdown文件中，作用是在当前文件引入`MathJax`引擎。

```javascript
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
```
&ensp;&ensp;&ensp;然后再使用`latex`方式编辑公式即可，用`$$公式$$`表示行间公式，本来latex中使用`\(公式\)`表示行内公式，但因为Markdown中\是转义字符，所以在Markdown中输入行内公式使用`\\(公式\\)`

## 问题

### 使用大括号

```latex
$$
\begin{cases}
\mu=\frac{1}{n}\sum_{i}x\_i\\\\
\sigma^2=\frac{1}{n}\sum_{i}(x\_i-\mu)^2
\end{cases}$$
```
$$
\begin{cases}
\mu=\frac{1}{n}\sum_{i}x\_i\\\\
\sigma^2=\frac{1}{n}\sum_{i}(x\_i-\mu)^2
\end{cases}$$
