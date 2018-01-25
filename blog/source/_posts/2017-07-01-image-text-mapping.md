---
title: 图文匹配论文阅读
date: 2017-07-01 15:35:18
tags:
  - papper
---

## 正文

&ensp;&ensp;&ensp;今天来一起阅读两篇图文匹配比较早起的论文：KCCA方法和Deep Fragment

<!--more-->

### KCCA

* [Framing Image Description as a Ranking Task: Data, Models and Evaluation Metrics (Extended Abstract) ](http://nlp.cs.illinois.edu/HockenmaierGroup/Papers/IJCAI2015/ExtendedAbstract.pdf)
&ensp;&ensp;&ensp; 提出了一种新的数据集PASCAL VOC-2008和Flickr 8K。其中前者有1000张图片，后者有8092张图片。每张图片对应五个对其的描述。提出一种基于KCCA的图像文本匹配检索模型。提出看待图像描述问题的新角度：检索角度，Ranking-based。提出一种评价Ranking-based system的评价标准，用以前推荐系统用的Recall@k评价。

### 基于检索

* [Deep Fragment Embeddings for Bidirectional Image Sentence Mapping](https://arxiv.org/pdf/1406.5679.pdf)

&ensp;&ensp;&ensp;文章提出了一种图像文本匹配的模型。不同于前面已有的直接将图像与文本映射到一个相同的空间中进行计算相似度。本文在更细的粒度上进行映射：即先将图像跟文本片段化，在将这些片段映射到一个共同的空间。在前面已有的ranking-objective基础上，本文设置两个ranking-objective，一个全局的一个局部片段的。
&ensp;&ensp;&ensp;本文的主要视角是图像的结构是非常复杂的，包含很多相互作用的物体。于是我们就将图片和文本分割成小片段得到他们之间隐含的关系。这里我们有一个基本的假设：图像对应的文本是本图片中包含的物体关系是很大的。而不是抽象到更高的层次。

&ensp;&ensp;&ensp;我们使用物体探测得到的物体作为图像片段，使用句子依赖树中的关系作为文本的片段。

![deepfragment](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/deepfragement.png?raw=true)

  * 文本片段化。
    我们希望将文本中描述的物体及其他们之间的关系抽取出来。
    所以我们使用句子依赖树，本文抛弃句子依赖树中的树结构，而使用树中没一个关系作为片段。
    如下图，将每一个词用词向量表示，然后用下面的式子将没一个关系依赖映射到嵌入空间。

    ![deepfragment-text-embed](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/deepfragment-text-embed.png?raw=true)

    其中w1表示词的one-hot向量，Wew1为词的词向量。WR和bR为每个关系的权重矩阵，s为关系最终在嵌入空间内的表示，s的维度可以通过交叉验证调节。

  * 图片片段化
    我们假设图片的描述都是基于图片中的物体和图片的整体上下文。那么很自然，图片的整体上下文和其中的物体就可以作为图片的fragements。
    通过RCNN网络得到物体，然后将每个物体和整张图片输入下面的式子映射到嵌入空间。
    ![deepfragment-image-embed](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/deepfragement-image-embed.png?raw=true[wkj]) 

  * 目标函数
    本文目标函数由两部分组成，一部分是全局目标函数，匹配的图像和文本的分数应该高于不匹配的，一部分是fragement层次上的目标函数，用于控制fragement之间的匹配。

    ![deep-objective](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/deep-objective.png?raw=true)

    其中theta是参数的表示，![theta](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/deep-objective2.png?raw=true)

    * 片段对齐目标函数fragement alignment objective
      直觉上就是句子中的一个片段如blue ball，必然在图像中有一个fragement与其对应且分数很高，其它图像中的片段与blue ball的匹配分数很低。
      其实这个规则很容易被打破
        * 句子的片段有可能不跟图片中跟任何片段相关。
        * 句子的片段所对应的图像中的物体可能没有被RCNN检测出来。
        * 其他图片中有这个片段但是与其对应的描述中忽略了对这个片段的描述。
      即使如此，这个规则在大多数情况下是能够很好使用的。
    下面就是fragement alignment objective:
    ![fragement-objective](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/fragement-objective.png?raw=true)
    上面式子中的求和是对训练集中的所有片段进行球和。ViTSj代表两段vi和sj的匹配分数。yij,代表片段vi和sj是否出现在同一个图像文本对中，如果出现则为1,否则为-1。

    而上面的目标函数假设fragement中的对应是稠密的，但是现实生活中往往不是这样的，比如下图中"boy playing"只对应一个图像fragement，于是我们使用多实例学习。

    ![f](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/mil2.png?raw=true)
    上图中代表负样例，绿色代表正样例，黄色代表在正样例包里的临时负样例。

    [MIL](http://blog.csdn.net/pkueecser/article/details/8274713)
    [MIL2](http://blog.csdn.net/carson2005/article/details/22499565)

    最终的目标函数：
    ![fragoj](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/mil.png?raw=true)
    其中是文本段j对应的正样例bag,mv(i)和ms(j)代表段vi和sj对应的图片和文本在训练数据的位置，其实最后一行表明的是yij只能取值-1和1，然后不对应于同一个图像-文本对的任意vi和sj段的yij值是-1。第一个等式的意思是保证每个正样例bag中至少有一个正样例。

    这个目标函数的意思就是为了防止漏掉一些没有检测到的文本fragement和正样例bag中的一些图像fragement的对应。让正样例bag中的fragement对应的yij不在是常量而是变量，然后取最小的目标函数至即可。

    * 全局目标函数

    Recall that the Global Ranking Objective ensures that the computed image-sentence similarities are consistent with the ground truth annotation.

    ![global](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/global.png?raw=true)
    其中gk代表第k张图像对应的fragement集合，gl则是第l个句子对应的fragement集合。
    用max截断是因为不应太小，再小也是负样本。

    ![global2](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/global2.png?raw=true)

    上面的全局目标函数使得匹配的图像文本Skk的分数比Skl和Slk的分数至少delta。

    * 实验
    ![experiment](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/experiment.png?raw=true)
