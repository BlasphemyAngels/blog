---
title: linux下制作动图
date: 2017-09-19 14:06:16
categories:
  - 工具
tags:
  - tools
  - 动图
---

## 正文

&ensp;&ensp;&ensp;动图是现今互联网上一种表示信息的重要形式，它可以用来表示工作流程、产品功能等。我在写markdown时常遇到一些描述操作流程的问题，这时使用单个图片是不足够的，而使用视频又面临加载慢的问题，很自然的就想到了用动图来解决这个问题。

<!--more-->

&ensp;&ensp;&ensp;废话少说，下面就讨论如何在linux制作动图。

### 录制视频

&ensp;&ensp;&ensp;动图是一个缩小版的的视频，所以要先录制视频，然后再转换为动图。

&ensp;&ensp;&ensp;我找到的比较简单使用的linux录制视频软件是`SimpleScreenRecorder`，具体安装如下：

* Arch Linux
```sh
sudo pacman -S SimpleScreenRecorder
```
* Ubuntu
```sh
sudo add-apt-repository ppa:maarten-baert/simplescreenrecorder
sudo apt-get update
sudo apt-get install simplescreenrecorder
```
&ensp;&ensp;&ensp;使用:

![simplescreenrecorder](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/simplescreenrecord.gif?raw=true)

### 转换为gif

&ensp;&ensp;&ensp;录制完视频后就要将视频转换为gif格式了，有很多转换工具，这里使用的是`ffmpeg`。

&ensp;&ensp;&ensp;安装：

* Arch Linux
```sh
sudo pacman -S ffmpeg
```
* Ubuntu
```sh
sudo apt-get install ffmpeg
```

&ensp;&ensp;&ensp;使用下面命令将录制好的视频转换为gif动图:

`ffmpeg -ss 00:00:20 -i input.flv -to 10 -r 10 -vf scale=200:-1 output.gif`

&ensp;&ensp;&ensp;参数：

* -ss 指定从视频的哪个地方开始转换
* -i 后面跟要操作的那个视频文件
* -to 指定到视频的哪个位置为止不再转换
* -r 帧速率，可以增大这个值输出更画质更优的GIF文件
* -vf 图形筛选器，GIF的缩放大小

### 合并多张动图

&ensp;&ensp;&ensp;有的时候需要将多张动图合并为一张，该怎么做呢？很简单，使用`convert`命令。

&ensp;&ensp;&ensp;参数：

* -delay value         每张图片直接的播放时间间隔
* -loop iterations     循环次数，0代表无限循环

&ensp;&ensp;&ensp;将要合并的多张动图放到一个文件夹中，然后执行：

`convert -delay 100 -loop 0 *.gif output.gif`

### 从图像序列生成动图

&ensp;&ensp;&ensp;有的时候我们有的是多张图片的序列，怎么将这些序列转化为动图呢？跟上面的命令相似，将图片序列放入到一个文件夹中，然后执行：

`convert -delay 100 -loop 0 *,gpg output.gif`

### 查看图像的大小

&ensp;&ensp;&ensp;`convert`还有一个应用就是查看图像的大小：

* `convert a.jpg -print "Size: %wx%h\n" /dev/null`

转载请注明:[Artemis的博客](https://BlasphemyAngels.github.io)--> [点此看原文](https://blasphemyangels.github.io/2017/09/19/2017-09-19-ffmpeggif/)
