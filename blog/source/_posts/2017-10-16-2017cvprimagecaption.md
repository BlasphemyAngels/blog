---
title: CVPR 2017 Image Caption论文阅读笔记
date: 2017-10-16 22:44:46
categories:
  - papper
tags:
  - papper
  - Image Caption
  - CVPR
---

## 正文
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

&ensp;&ensp;&ensp;最近在看cvpr2017的papper，看到了一些有关图像描述的好文章，读了一下，做个笔记。

<!--more-->

### [Attend to You: Personalized Image Captioningwith Context Sequence Memory Networks](https://arxiv.org/abs/1704.06485)

&ensp;&ensp;&ensp;亮点在于亮点：
* 个性化图像描述
* 使用上下文序列记忆网络进行学习，而非RNNs，LSTMs

&ensp;&ensp;&ensp;文章指出了两个应用，一个是在社交网络中，人们拍了照片要发图的时候，能够自动生成描述，二是自动生成标签。
&ensp;&ensp;&ensp;文章使用一个记忆仓库存储多种类型的上下文信息，每次将捕捉到的单词加入到记忆中，保留长时间信息，随后使用记忆生成下一个词。这样避免了因梯度消失而引起的信息损失。随后使用CNN记忆架构学习周围信息进行上下文理解。

#### Context Sequence Memory Network(CSMN)
&ensp;&ensp;&ensp;CSMN网络架构如下：
![csmn](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/csmn.png?raw=true)

&ensp;&ensp;&ensp;模型建立了一个记忆仓库，记忆仓库中含有三种类型的记忆：
* 图像记忆
* 用户偏好记忆
* 序列历史记忆

&ensp;&ensp;&ensp;如上图中的a图，将图像先用CNN处理，取CNN中的某一层数据作为图像特征，然后将图像特征乘以一个向量W即可得到最终图像记忆，向量W是可训练的参数。而用户偏好的计算是使用`TF-IDF`算法得到用户的前`TOP-K`个单词。然后用`one-hot`向量表示这些单词，再乘以一个向量W即得到的向量就是用户偏好记忆。同样向量W是可训练的参数。

&ensp;&ensp;&ensp;而序列历史记忆则是将前面预测到的单词连接在一起即可。如图中\\(y\_1,...,y\_{t-1}\\)就是序列历史记忆。每次生成新单词后会将其加入到序列历史记忆中。

&ensp;&ensp;&ensp;可以发现每种记忆一共训练了两份，分别以`a`和`c`为下标。每份记忆训练的W也是不同的，所以最终得到的两种记忆有可能是不同的（因为训练得到的`W`是不同的）。

&ensp;&ensp;&ensp;那么训练的两份记忆是用来干啥的呢？下面一边介绍模型的预测流程一边说明：

&ensp;&ensp;&ensp;先将上一次预测到的单词经过一个线性变换得到一个查询\\(q\_t\\)，然后使用使用查询与第一份记忆做点乘并做`softmax`操作，这样就得到了记忆注意力向量\\(p\_t\\)，\\(p\_t\\)的作用是选择哪些记忆会对下一个单词的产生起效果。将\\(p\_t\\)与第二份记忆相乘即可得到能对下一个单词的产生起作用的记忆。随后对得到的记忆进行卷积操作，通过使用不同的卷积核，我们能够通过卷积得到不同的记忆单元的组合表示。随后通过线性变换和`softmax`即可得到下一个单词的概率。每次产生新的单词会更新序列历史记忆。

&ensp;&ensp;&ensp;对于hashtag的预测，我们只需要去掉word output memory即可。

#### 为什么记忆CNN有用

&ensp;&ensp;&ensp;尽管卷积记忆网络不能对结构化的时序数据建模，除非加入时序信息。但是文章提出的网络有所不同，序列记忆能够使得CNN捕捉时序信息。用户偏好记忆可以使得CNN能够正确的捕捉上下文单词的重要性。例如用户当前相关的单词有`fashion`、`street`和`landscape`，如果`fashion`在用户偏好记忆的最上面，而`street`在与`fashion`相关的词旁边，那么用户的偏好可以被认为是：`street fashion`，而如果`landscape`在最上面，`street`在`landscape`相关词周围，那么用户偏好被认为是`landscape`。如果不使用记忆CNN，那么`street`的这两种用法是很难区分的。

#### 实验

![exp](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/attenp_exp.png?raw=true)

### 
