---
title: Google develop days 2018 之旅
date: 2018-09-21 18:14:19
tags: 
- Google
- Google develop days
- 展会
categories: 
- Google
- Meeting
toc: true
---
经过两天 Google Develop Day 之旅，强烈感受到谷歌的奉献精神，与国内其他大会不同，它更注重开发者和用户的意见，一直在为开发者提供便捷开发工具和接口的同时，也在为提高交互流畅性和用户体验度的道路上努力前进着。我将针对 Google 大会中对于开发者而言比较重要的几个部分，结合个人看法简单说明一下大会中的主要内容。

#### Android JetPack

Google 在本次开发者盛会中，依旧将 Android 作为主要的分享主题，而 Android JetPack 也是作为本次 Android 分会馆中比较重要的一部分。正如本次大会介绍的那样，JetPack 作为 “Android开发加速器”，它是一系列的组件、工具和架构指南。主要从四个方面提升 Android 开发的效率：<!--more-->

- 架构：
  - WorkManager：后台任务执行
  - Navigation：页面之间跳转
  - Paging：数据分页功能
  - Data Binding：数据绑定
  - Lifecycles：生命周期处理
  - LiveData：数据与UI实时更新
  - Room：数据缓存
  - ViewModel：UI与数据管理
- 行为：
  - Slices：简化交互和操作
  - Download Manager：内置下载器
  - Media&Playback：多媒体
  - Permissions：权限申请
  - Notifications：通知
  - Sharing：共享
- 界面：
  - Fragment
  - Layout
  - Palette
  - Animation & Translations
  - Auto，TV & Wear
  - Emoji
- 基础：
  - Kotlin Extensions
  - AppCompat
  - Multidex
  - Test

JetPack也提供了向后兼容的特性，能让较低sdk版本的应用依旧可以享受这些工具组件带来的便利。



#### FireBase

什么是 FireBase？FireBase 是一个云端数据存储平台，可以轻松实现数据的实时更新。它旨在轻松打造富有吸引力的app，帮助了解用户行为以及诊断和监控 app 的表现，然后根据用户的操作来定位用户范围。

![image](http://upload-images.jianshu.io/upload_images/5256969-a6a8ae9cf57caff5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- ML Kit 将谷歌机器学习方面的成果带给移动应用开发者：

![image](http://upload-images.jianshu.io/upload_images/5256969-eb7c088276ea0f54.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- 数据分析能力：

通过机器学习，并动态绑定训练模型，应用不需要升级就可以更新模型。

- 崩溃信息采集：

可以通过 Crashlytics 提供的 sdk 可以将应用的一些崩溃信息采集到FireBase上面。

- 更多：

  ![image](http://upload-images.jianshu.io/upload_images/5256969-d1d9bd7bb21587c4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ​

#### Flutter

Flutter 可以媲美原生的交互体验，相对于 react native 来说，绘制过程省去了一个环节，能够更加快速的渲染。同时也支持与原生 Android 和 iOS 业务功能的代码混编。

![image](http://upload-images.jianshu.io/upload_images/5256969-b841a95fccb33755.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



为什么使用 Flutter？

- 可自定义的流畅页面
- 可同时在 Android 和 iOS 设备上进行原生编译
- 高效的开发工具，如 hot load(热重载)
- 响应式编程
- 各种快捷实用的插件（imagepicker，设置滤镜，社交平台分享等，比原生更加方便）



#### ARCore

应用场景：

购物、AR特效、游戏、社交

![image](http://upload-images.jianshu.io/upload_images/5256969-deab349fbe57b17d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

主要功能：

- 运动跟踪

  ARCore 利用摄像头图像中的视觉差异特征并结合设备中惯性传感器的测量结果，估算设备的位置和方向。

- 光估测

  ARCore 可以检测其环境光线的相关信息，并提供给定摄像头图像的平均光强度和色彩校正，达到能与周围环境相同的光照来照亮虚拟物体，提高它们的真实感。

- 环境理解

  ARCore通过检测特征点来检测平面，并让这些平面可以由我们的应用来运作平面。



同时，ARCore可以通过云锚点将锚点和附近特征点发送至云端托管，然后可以与其他用户进行环境共享和交互。



#### TensorFlow

TensorFlow 是目前世界上最热门的机器学习类开源框架。它可以让模型通过不断的机器学习来产生一套规则，来达到我们预期的效果，进而来处理大量数据，代替人工处理的成本。