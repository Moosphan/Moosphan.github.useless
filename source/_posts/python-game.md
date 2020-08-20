---
title: Android版微信跳一跳小游戏如何利用技术手段达到高分！
date: 2018-01-01 22:31:20
tags: 
- 微信小游戏
- Python
- 游戏脚本
categories: 微信小游戏
toc: true
---
本文主要来讲个个好玩的东西，近来微信刚出的跳一跳微信小程序的游戏很火，看到很多人都达到了二三百分就各种刷朋友圈了。
![甩手一个表情](http://upload-images.jianshu.io/upload_images/5256969-429ad8341f7a4759.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终我们达到的分数却是这样的：


![羡慕吧](http://upload-images.jianshu.io/upload_images/5256969-163bb346d8a6d114.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
一定会有人拍手叫好，“黄金右手”！说真的，我已经不用右手好多年，而且我玩这个游戏压根就没用手，而是意念！哈哈，别喷我，开个玩笑而已，肯定是利用技术手段啦，什么技术？python喽～哈哈，不过不是我写的，我自己是做Android开发的，我对于python从来没有接触，只是恰好在蛋哥公众号看到关于这个游戏的文章，觉得有意思，就花了点时间试着搞了一下，没想到还跑成功了，收获也挺多的😄。本文针对mac系统+Android全面屏手机，如想了解其他系统或者手机的教程，可以到项目主页或者issue中探索一下。好了，下面给大家看看它的“庐山真面目”。

关于游戏脚本，作者已经开源在了github，地址在https://github.com/wangshub/wechat_jump_game，有兴趣的可以先去看看。
一般的，手机型号比较老（像Android4.3以下的），或者比较新（像vivox20，小米mix2等等刚出的全面屏手机），再或者比较稀有的（像TCL，联想，诺基亚等等），顺利的跑起脚本还是挺难的，多少都会有些问题，由于脚本是作者几个小时就写好的，已经很厉害了，适配的这些问题不可能全部都面面具到，毕竟Android机型千千万呀！希望大家多多体谅，作者精力也是有限，不可能及时回答每一个人的问题，大家毕竟也是搞技术的，有些问题应该都有能力解决的，例如issue，度娘等等。这里我简单说明以下自己用mac和vivox20如何跑起来的。

首先，我们到作者的源码地址看看原理和Android的使用步骤,如下：
>将手机点击到《跳一跳》小程序界面；
用 ADB 工具获取当前手机截图，并用 ADB 将截图 pull 上来
```
    adb shell screencap -p /sdcard/autojump.png
    adb pull /sdcard/autojump.png .
```
>计算按压时间
手动版：用 Matplotlib 显示截图，用鼠标点击起始点和目标位置，计算像素距离；
自动版：靠棋子的颜色来识别棋子，靠底色和方块的色差来识别棋盘；
用 ADB 工具点击屏幕蓄力一跳；
```
    adb shell input swipe x y x y time(ms)
```
原来是利用adb来计算和模拟位置的，我们暂时不需要关心这个，再来看看Android手机使用步骤：
- 安卓手机打开 USB 调试，设置》开发者选项》USB 调试
- 电脑与手机 USB 线连接，确保执行adb devices可以找到设备 ID
- 界面转至微信跳一跳游戏，点击开始游戏
- 运行python wechat_jump_auto.py，如果手机界面显示 USB 授权，请点击确认
- 请按照你的手机分辨率从./config/文件夹找到相应的配置，拷贝到 *.py 同级目录./config.json（如果屏幕分辨率能成功探测，会直接调用 config 目录的配置，不需要复制）

OK，我就按照步骤一步一步来：
1. 打开手机开发者选项和usb调试，这一步我想不需要多说了，大家应该都知道怎么做；
2. 需要确保adb devices可以找到设备。
>搞移动端开发的应该都知道adb吧，不过可能有些人没有接触过，这里就简单说明一下如何执行adb命令。首先需要下载adb工具，一般Android studio的sdk中自带了，我们只需要配置一下环境变量就可以了，想知道如何配置，可以遵循如下步骤：
```
- 打开mac的terminal终端，输入 cd ~/ 【进入当前用户的home目录】
- 输入 touch .bash_profile 【如果没有.bash_profile这个文件，则创建一个这个文件】
- 输入 open .bash_profile 【打开我们创建的这个文件，此时应该弹出一个文本编辑框，如果是第一次配置环境，那么文本编辑框为空白】
- 在打开的文本编辑器中写入如下代码：
   export ANDROID_HOME=/usr/local/opt/android-sdk
   export PATH=${PATH}:${ANDROID_HOME}/tools
   export PATH=${PATH}:${ANDROID_HOME}/platform-tools
- 注意的ANDROID_HOME后面应该根据自己的sdk路径来填写，其余可以直接复制。至于sdk路径，可以打开Android Studio，在preference(Windows的setting)中搜索sdk来查看。
在终端中输入 source .bash_profile 【使我们的改动生效】
- 输入 adb 【验证是否完成配置，如果不显示 adb: command not found，说明配置完成 】
```
如果没有用过Android studio，那么可以去百度一下如何安装，我相信这对于大家来说不是一件困难的事，安装完成后只需要按照上面说的配置一下环境变量就可以了。接下来我们将手机连接到电脑，并开启第一步中的设置选项后，在电脑终端输入：
```
adb devices
```
不出意外的话，终端会出现类似如下内容：
```
Last login: Mon Jan  1 20:20:11 on ttys000
MoosdeMacBook-Pro:~ moos$ adb devices
List of devices attached
a619aaxx	device
```
这样就代表我们adb设备连接成功了。

3. 打开我们的微信中“跳一跳”游戏小程序，点击开始游戏，手机出现游戏初始界面；
4. 要求我们运行脚本项目中的python文件，这就需要我们安装python了，不用担心，一般mac系统自带了python，我们终端输入如下命令：
```
 python
```
>如果出现如下内容，则说明我们已经安装过了：
```
MoosdeMacBook-Pro:~ moos$ python
Python 2.7.10 (default, Jul 15 2017, 17:16:57) 
[GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.31)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
>如果没有安装python，可以参考该文章：http://blog.csdn.net/u010828718/article/details/70257622
下面我们就需要运行wechat_jump_auto.py这个脚本文件了，这个是自动玩游戏，你也可以选择手动玩，运行wechat_jump_py3.py即可，如何运行？很简单，只需要把github上面的项目下载下来，并进入到该目录下，执行如下命令即可（以自动版为例）：
```
 python wechat_jump_auto.py
```
这样，我们就可以跑起来这个脚本了，但是你可能会遇到这个问题：
```
Traceback (most recent call last):
  File "wechat_jump_auto.py", line 8, in <module>
    from PIL import Image, ImageDraw
ImportError: No module named PIL
MoosdeMacBook-Pro:wechat_jump_game-master moos$ 
```
只需要安装pillow，执行如下命令即可：
```
 sudo pip install Pillow
```
记得加上sudo，需要获取系统权限。

这样，应该基本可以跑起来脚本了。但是，vivox20手机运行了脚本之后，一直没有自动游戏，后来发现，可能是不同手机分辨率和尺寸差异，导致脚本没有是识别到对应的模拟按压的坐标位置，可以修改一下对应的按压参数为320，1210，720，910，对应的修改位置是wechat_jump_auto.py中的如下参数：
```
swipe['x1'], swipe['y1'], swipe['x2'], swipe['y2'] = 320, 410, 320, 410
```
该修改意见已经被作者合并到主分支了，打开该文件就可以看到了。再跑一下试试，发现还是不行，程序在运行，位置坐标也在变化，但游戏没有进行，那可能就是手机的问题了，尝试开启开发者设置中的usb安全验证设置，我再跑，嘿，可以了：
![效果图](http://upload-images.jianshu.io/upload_images/5256969-1d44bdc2bf04fad3.jpg?imageMogr2/auto-orient/strip)请忽略这渣图，vysor还没有很好的适配vivox20，并不是gif的问题😢，再看看终端的数据：
```
('scan_start_y: ', 720)
(1514815959, 0, 0, 0, 0)
adb shell input swipe 320 1210 720 910 200
('scan_start_y: ', 670)
(1514815962, 0, 0, 0, 0)
adb shell input swipe 320 1210 720 910 200
('scan_start_y: ', 670)
(1514815966, 0, 0, 0, 0)
adb shell input swipe 320 1210 720 910 200
('scan_start_y: ', 820)
(1514815971, 338, 1224, 788, 968)
adb shell input swipe 320 1210 720 910 710
('scan_start_y: ', 920)
(1514815976, 697, 1203, 400, 1008)
adb shell input swipe 320 1210 720 910 487
('scan_start_y: ', 820)
(1514815979, 320, 1275, 839, 940)
adb shell input swipe 320 1210 720 910 847
('scan_start_y: ', 870)
(1514815984, 392, 1194, 718, 1009)
adb shell input swipe 320 1210 720 910 514
('scan_start_y: ', 870)
(1514815987, 660, 1167, 450, 1052)
adb shell input swipe 320 1210 720 910 328
('scan_start_y: ', 770)
...
```
这样就没毛病了，同时，我还修改了2160x1080的配置参数，提高了跳跃的准确度，达到几千分不是问题，并且已经被作者同意了合并了，无需再做额外修改了。

在借用该脚本作者的一句话：
```
事实证明，机器人比人更会玩儿游戏。
```
好了，大家是不是已经迫不及待想去刷分了呢😄，不过分高虽好，可不要“贪杯”哟，会没朋友的。
