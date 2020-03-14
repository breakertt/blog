title: 车牌识别
date: 2018/10/14 12:28:30
categories:
- 编程
tags:
- 计算机视觉
- Python
thumbnail: https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/2018/1014/1014bg_output.jpg
toc: true
---

一筹莫展。

<!-- more -->

## Part 1
我一开始是想自己去训练这个车牌识别的的 这是Part1的主要内容

### 生成车牌
为什么要生成车牌，因为车牌和人脸不一样，车牌的数据比较隐私，国内并没有说有任何公开的 可以拿来训练车牌的数据集，也没有时间慢慢去收集、标注，只能自己来生成。
[end-to-end-for-chinese-plate-recognition](https://github.com/szad670401/end-to-end-for-chinese-plate-recognition)
这个项目里面的"genplate.py"可以模拟生成车牌并附带坐标信息.

### 判断车牌位置
用FasterRcnn训练模型，结果还可以

### 问题
框出车牌之后不知道如何训练了Orz 具体的字符识别和车牌矫正都没有比较好的数据。

## Part 2
然后我就去找了一些开源项目，主要是EasyPR 和 HyperLPR。考量之后是HyperLPR比较好。
https://github.com/zeusees/HyperLPR
使用方法也非常简单，运行demo.py就可以了。


## 参考文献:
文中提到的两个github项目的文档