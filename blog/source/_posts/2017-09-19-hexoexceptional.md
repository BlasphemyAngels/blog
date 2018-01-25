---
title: hexo配置打赏功能
date: 2017-09-19 10:44:32
categories:
  - 博客
tags:
  - hexo
  - 打赏
---

## 正文

&ensp;&ensp;&ensp;现在是互联网的世界，越来越多的互联网文章出现，人们的忙碌使得他们往往借助互联网寻求解决问题的方法。碎片化文字和博客的出现使得人们解决问题的方式更加简单。由此，往往对于一些优秀的博客，我们常常希望给他一定的鼓励以希望他继续创作，基于此，打赏功能应运而生。

<!--more-->

&ensp;&ensp;&ensp;本人使用的是`hexo+next`构建博客，因此下面就讲解一下在`hexo next`下进行打赏功能的配置。

### 生成二维码

&ensp;&ensp;&ensp;使用微信和支付宝生成相应的收款码分别保存为`wechat.png`和`alipay.png`

&ensp;&ensp;&ensp;将生成的二维码照片放入到`themes`下的`hexo-next`主题目录下的`source/images/`文件夹下

### 配置

&ensp;&ensp;&ensp;进入`next`主题目录，编辑主题的`_config.yml`，加入如下代码（如果已经存在，则修改成跟如下代码一致）:

```styl
reward_comment: 您的支持将鼓励我继续创作！
wechatpay: /images/wechat.png$
alipay: /images/alipay.png$
```

### 取消闪烁效果

&ensp;&ensp;&ensp;如下图，当将鼠标放到二维码图片上时，图片下面的字会闪烁，在我看来，这不是一个好的效果，于是决定取消它（如果你喜欢的话可以不取消）。

![gif](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/exceptional.gif?raw=true)

&ensp;&ensp;&ensp;如何取消呢？只需将主题目录下的`source/css/_common/components/post/post-reward.styl`文件中的下面代码注释掉即可。

```styl
#wechat:hover p{$
    animation: roll 0.1s infinite linear;$
    -webkit-animation: roll 0.1s infinite linear;$
    -moz-animation: roll 0.1s infinite linear;$
}$
#alipay:hover p{$
    animation: roll 0.1s infinite linear;$
    -webkit-animation: roll 0.1s infinite linear;$
    -moz-animation: roll 0.1s infinite linear;$
}$
```


转载请注明:[Artemis的博客](https://blasphemyangels.github.io)--> [点此看原文](https://blasphemyangels.github.io/2017/09/19/2017-09-19-hexoexceptional)
