title: FaceNet使用记录 - 我的人脸识别学习笔记02
date: 2018/6/19 11:30:30
categories:
- 编程
tags:
- 人脸识别
- 计算机视觉
- Python
thumbnail: https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/20200314000042.png
toc: true
---

即使是用别人的东西也花了好大的气力。

<!-- more -->

首先贴上FaceNet的github [davidsandberg/facenet](https://github.com/davidsandberg/facenet) [论文](https://www.cv-foundation.org/openaccess/content_cvpr_2015/papers/Schroff_FaceNet_A_Unified_2015_CVPR_paper.pdf)

## 为什么是FaceNet?
1. 机器环境
老板给的机器已经配好了 TensorFlow + Cuda, FaceNet所需的环境正好是这个 √
2. 社区资源
Github上使用FaceNet进行二次开发的样例非常多
例如:[shanren7](https://github.com/shanren7/real_time_face_recognition) [bearsprogrammer](https://github.com/bearsprogrammer/real-time-deep-face-recognition/)
3. 识别率和预处理模型的提供
FaceNet提供的预处理模型在LFW测试数据集上的准确度已经达到了99.65%,并且提供了分别以 CASIA-WebFace 和 VGGFace2 为训练集的预处理模型.
4. 功能性
人脸识别主要有两块，一个是对脸的识别，另一个是对人物的ID的识别.FaceNet在这两个功能上都有很好的完成度


## FaceNet的基本使用(此节主要借鉴facenet的[wiki](https://github.com/davidsandberg/facenet/wiki/Train-a-classifier-on-own-images))
此处已经默认装好了相关Python库以及Tensorflow
### 1. 克隆 FaceNet 的 Github 库
```powershell
git clone https://github.com/davidsandberg/facenet.git
```


### 2. 比对两个人脸的欧式距离
FaceNet是采用CNN神经网络将人脸图像映射到128维的欧几里得空间，我们可以根据两幅人像的欧几里得距离去判断两个人像的相似程度。两个人像之间的欧几里得距离越近，说明它们越相似。一般欧式距离小于1,就可以认为是同一个人。
![None](https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/2018/0619/999009-20170305161101282-1344836451.png)
```powershell
python compare.py models\20170511-185253\20170511-185253 Cate_Blanchett_0001.png  Cate_Blanchett_0002.png
```


### 3. 训练自己的数据
#### 训练集的结构
训练集的结构如下，每个人都有独立的文件夹
```Aaron_Eckhart
    Aaron_Eckhart_0001.jpg

Aaron_Guiel
    Aaron_Guiel_0001.jpg

Aaron_Patterson
    Aaron_Patterson_0001.jpg

Aaron_Peirsol
    Aaron_Peirsol_0001.jpg
    Aaron_Peirsol_0002.jpg
    Aaron_Peirsol_0003.jpg
    Aaron_Peirsol_0004.jpg
    ...
```
#### 剪切出人脸
FaceNet 提供了使用 MTCNN 对齐人脸的脚本 代码如下
```powershell
python src/align/align_dataset_mtcnn.py \
~/datasets/my_dataset/origin \
~/datasets/my_dataset/train \
--image_size 182 \
--margin 44
```
加速多线程版：
```powershell
for N in {1..4}; do \
python src/align/align_dataset_mtcnn.py \
~/datasets/my_dataset/origin \
~/datasets/my_dataset/train \
--image_size 182 \
--margin 44 \
--random_order \
--gpu_memory_fraction 0.25 \
& done
```
#### 训练.pkl
```powershell
python src/classifier.py TRAIN ~/datasets/my_dataset/train/ ~/models/model-20170216-091149.pb ~/models/my_classifier.pkl --batch_size 1000
```
#### 测试
```powershell
python src/classifier.py CLASSIFY ~/datasets/my_dataset/test/ ~/models/model-20170216-091149.pb ~/models/my_classifier.pkl --batch_size 1000
```

## 后记
既然已经大概了解FaceNet怎么用了，就要开始真正的魔改应用之路了！下回见分晓。

## 参考文献:
1. [谷歌人脸识别系统FaceNet解析](https://zhuanlan.zhihu.com/p/24837264)
2. [史上最全的FaceNet源码使用方法和讲解（一）（附预训练模型下载）](https://blog.csdn.net/u013044310/article/details/79556099)
3. [人脸识别（Facenet）](https://blog.csdn.net/xingwei_09/article/details/79161931)
4. [FaceNet---深度学习与人脸识别的二次结合](https://www.cnblogs.com/xiaohuahua108/p/6505756.html)