title: 使用AegiSub和FFmpeg为视频打上字幕(啊？Pr是什么？)
date: 2018/10/16 13:41:30
categories:
- Media
tags:
- FFmpeg
- VideoTech
thumbnail: https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/20200314000227.png
toc: true
---

SESA干培。

<!-- more -->

## 工具的准备
### 1. **Aegisub**
请访问[Aegisub官网](http://www.aegisub.org/)对应系统进行下载以及安装
### 2. **FFMpeg**
Windows用户:
* 下载FFMpeg: https://ffmpeg.zeranoe.com/builds/ (请选择static版本)
* 解压FFmpeg: 把下载好的压缩包解压到C:\ffmpeg目录下
* 配置环境变量: 按照[这篇文章](https://jingyan.baidu.com/article/b24f6c82cba6dc86bfe5da9f.html)将刚才的目录添加入Path
* 测试运行: 按WIN+R,输入CMD,在跳出来的终端窗口中输入```ffmpeg -version``` 如有信息则说明安装成功

MAC用户:
* 下载安装homebrew: 打开Termial,输入```/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"``` 
* 下载安装FFMpeg: 继续在Termial中输入 ```brew install ffmpeg --with-fdk-aac --with-ffplay --with-freetype --with-libass --with-libquvi --with-libvorbis --with-libvpx --with-opus --with-x265```
* 测试运行: 继续Terminal中输入```ffmpeg -version``` 如有信息则说明安装成功

## 打时间轴,得到到字幕文件
请参考以下视频进行相关学习,以懂得如何为文本配上时间轴并得到.ass字幕文件为目标.样式和特效如果自己有感兴趣的话可以深究一下,这是一个无底洞.
https://www.bilibili.com/video/av2345646/
https://www.bilibili.com/video/av32482188/

## 把字幕压进视频轨
用cd命令将终端位置调到视频文件夹,下面是一个例子.
```cd C:\Users\break\Videos```
用下面这个命令把字幕烧进视频流,其中文件名按照你的文件名进行改动.
```ffmpeg -i mymovie.mp4 -vf ass=subtitles.ass mysubtitledmovie.mp4```
如果遇到了什么问题,请打开参考文献[4]看一下其中有没有提到,如果有很奇怪的问题欢迎[Email我](mailto://admin@ib32.com).

## 参考文献:
1. [FFmpeg安装（windows环境）](https://www.cnblogs.com/xiezhidong/p/6924775.html)
2. [在mac os下使用FFmpeg](https://www.jianshu.com/p/0b1c98a28fd4)
3. [Use ffmpeg to add text subtitles - StackOverflow](https://stackoverflow.com/questions/8672809/use-ffmpeg-to-add-text-subtitles)
4. [使用FFmpeg将字幕文件集成到视频文件](http://www.yaosansi.com/post/ffmpeg-burn-subtitles-into-video/)
