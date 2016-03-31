---
layout: post
title: Android-ANR实践
published: true
tags: [ANR]
---

ANR，也就是我们所说的Application Not Responding，应用无响应。

一般就是主线程阻塞导致用户交互，UI展示或者其他需要在主线程处理的消息未及时处理。

怎么避免就不说了，本文主要说下怎么处理及记录一下自己遇到的案例。

## ANR日志

android会在应用ANR之后在`/data/anr`路径下生成一个trace文件，该文件详细展示了ANR相关的信息包括但不限于ANR进程、包名、堆存储信息、线程名及状态、ANR引起的原因。

## 获取日志

只需要命令行`adb pull /data/anr/`即可。trance文件会拉取到你的PC的根目录下（具体看cmd信息）。

## 实践分析

> 从今天开始遇到ANR分析的案例再更新这里。

### 20160331

今天遇到一个ANR，分析了一下日志，找到如下信息：

![android-anr-case-1.png]()

这个很清楚AutoScrollHandler在处理消息时调用了ViewPager的setCurrentItem，导致无响应，我们知道这个API在一下子跳转太多的item会生成同样数量的View导致耗时，到此问题解决。

