title: 魔改(二次开发)FaceNet为己用 - 我的人脸识别学习笔记03 - 完结篇
date: 2018/6/22 17:28:30
categories:
- 编程
tags:
- 人脸识别
- Python
- 计算机视觉
thumbnail: https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/2018/0621/0621bg_output.jpg
toc: true
---

魔改即是正义！

<!-- more -->
{% meting "474574924" "netease" "song" "autoplay"%}
*本文写于年轻人(也就是我)的第一次加班(谁让我在下班前开始了神经网络的训练呢)*

我的魔改成果 [face_recognition_with_TCPsocket](https://github.com/imaginebreake/face_recognition_with_TCPsocket)
顺便上个逻辑图 ![None](https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/2018/0621/FRWTS.png)

## 寻找志同道合者并沆瀣一气(不是
根据学习笔记02，我已经对FaceNet的使用有了初步的了解。FaceNet自带的Demo用来做人脸数据库并识别人脸已经是很完善的了，所以需要做的改动并没有想象的那么多。我第一步要达到的是能在本地提取一个人并输入照片的时候可以返回框出人脸并带上人名的图像，基于这个目的，我去Github上搜索了一些同样是基于FaceNet二次开发的项目。

首先找到的是shanren7的一个实时人脸识别的项目，这个项目和我需要的基本一致，但是问题有二:
1. 只能输出一个人QAQ
2. 用的预训练模型并不是davidsandberg/facenet内提供的

然后我继续寻找，找到了借鉴了shanren7项目开发的另一个项目[bearsprogrammer/real-time-deep-face-recognition](https://github.com/bearsprogrammer/real-time-deep-face-recognition/).这个项目正好解决了前一个提到的两个问题，但是同时也出现了一些其他的问题.
1. 开发者用Python程序内定义好的list来储存人名，要实时更新并不方便。
2. 这个系统是纯本地的，但是我要做的系统是可以远程训练和识别的。

其中问题一其实只是一个小问题，很快就能搞定；而问题二基本上是开发一个新功能了，工作量比较大，也摸了好几天。

需要一提的是，这两个项目的主要的Detect的程序也是由FaceNet的demo修改而来。

那接下来就来对这个项目进行进一步的开发！

## 问题一
对于问题一的解决，在数据量不大的时候最好的方案就是把这个list单独搞一个文本文件，在每次识别的时候读取为list(数据量上去就要用数据库了)。选择有两个，json和csv，因为我这个字段只有一个编号和人名，用csv比较清爽所以就这么决定了。Python对csv的调用非常简单了，代码如下:

**读取**
```python
with open("./models/human_name.csv", "r") as f:
    human_name_reader = csv.reader(f)
    HumanNames = [row[1] for row in human_name_reader]
    del HumanNames[0]
```
**追加**
```python
with open("./models/human_name.csv", 'a+') as f:
    f.write(str(count).zfill(6) + "," + name + "\n")
```
问题一就这样解决了!

## 问题二
### 远程检测
在我把问题一解决之后，老板说:那你给这个项目加个Socket，然后我就一脸懵逼:Socket是啥???然后我就去百度了一波。Socket其实是一种对TCP/IP的封装，要给我的项目加上Socket，也就是加上通过网络检测和训练的功能。C我是不太熟的，Python也有现成的Socket库，那就用Python了。
关于Python上Socket的编写，我很大程度上参考了简书上的[一篇文章](https://www.jianshu.com/p/2a4b859e05df),同样都是传输图像。
啃了一个下午，总算是把检测部分的代码搞了出来，并在本机检测通过，中间有几个我觉得值得一提的小波折。

1.图像文件之后发送过去之后,文件体积一致,但是文件头里面多了一些之前的传输信息,花了一个小时debug总算是找到了问题所在。
**修改前**
```python
while True:
    data = sock.recv(SIZE)
    if not data :
        print 'reach the end of file'
        break
    elif data == 'begin to send':
        print 'create file'
        checkFile()
        with open("./mydataset/test/test.jpg", "wb") as f:
            pass
    else:
        with open("./mydataset/test/test.jpg", "ab") as f:
            f.write(data)
```
**修改后**
```python
while True:
    data = connection.recv(SIZE)
    if not data :
        print("reach the end of file")
        break
    elif data == 'begin to send':
        print("create file")
        checkFile()
        with open("./mydataset/test/test.jpg", "wb") as f:
            data = None
            pass
    else:
        with open("./mydataset/test/test.jpg", "ab") as f:
            f.write(data)
```
问题就在于没有对socket接收的缓存数据进行清空

2.Ubuntu的防火墙:要现在本机的防火墙上先开放要使用的端口

本机测试通过了,我就开始拿笔记本远程调试,中间出现了一些因为python版本导致的问题，就不具体说了，[请看此](https://blog.csdn.net/yexiaohhjk/article/details/68066843),解决这个问题之后程序跑的十分顺利。
### 远程登记人脸
我的想法是调用电脑的摄像头存下一系列照片然后打包成.zip发送给服务器，然后进行人脸对齐和特征的储存。中间基本上没遇到什么太大的问题。
### 优化
之前无论哪个操作，服务器端都是直接用Python调用命令，这样每次都是要重新打开一个TensorFlow的Session，耗时很不乐观。所以我决定人脸检测这部分一直使用同一个Session，训练的时候再重开另一个Session。
这样就会出现一个问题，就是每次人脸登记完之后要重新加载包含人名的.csv和包含人脸特征的.pkls,所以我在代码中添加了一个update的变量让程序知道是否要重新加载。
为了更好的交互，我第一次接触了Visual Studio并配好了Opencv，步履维艰的写下了自己人生中第一个C++程序，想必以后还有的是打交道的机会。这个C++的控制台小程序可以调用client_dete.py和client_train.py。

## 后记
[Github](https://github.com/imaginebreake)里面总算是有了个不少是自己写的代码的项目，完结撒花！

## 参考文献:
1. [shanren7/real_time_face_recognition](https://github.com/shanren7/real_time_face_recognition)
2. [bearsprogrammer/real-time-deep-face-recognition](https://github.com/bearsprogrammer/real-time-deep-face-recognition/)
3. [TCP/IP、Http、Socket的区别](https://blog.csdn.net/Pk_zsq/article/details/6087367)
4. [[毕设记录] python利用socket进行文件传输](https://www.jianshu.com/p/2a4b859e05df)