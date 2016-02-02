---
layout: post
title: Android中TraceView的使用
published: true
tags: [测试,TraceView,性能]
---

Android性能优化系列的第一篇文章，traceview在我们开发中扮演了一个重要的角色，项目积累到一定程度之后就开始出现卡顿设置ANR，那么分析导致这些问题原因就至关重要了，TraceView就是做这样一个工作的工具。

*正好工作中需要分析一个ANR的问题，以前一直没写下来，这次就顺便就记录一下。*

## 找到工具

手机连接电脑，然后在DDMS里找到device面板如图：

![device-panel](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-TraceView%E7%9A%84%E4%BD%BF%E7%94%A8/device-panel.png)

选中要分析的进程，点击Start Method Profiling（红框标出），弹出的执行选项选择样本分析（默认即可，我喜欢把样本的时间间隔设置为500），然后手机上执行你觉得卡顿的操作，执行完之后点击Stop Method Profiling（还是刚才的按钮）。

## 开始生成

这时候TraceView会为你生成一个trace文件，一般来说Eclipse里你停止分析之后就会自动弹出来的，trace分为上下两部分，上部分是时间轴面板，下面是分析面板，对我们来说分析面板比较重要，如图：

![traceview](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-TraceView%E7%9A%84%E4%BD%BF%E7%94%A8/traceview.png)

这个表格中的每一列是什么意思呢？如下表

|列名|描述|
|---|---|
|Name|该线程运行过程中所调用的函数名|
|Incl Cpu Time|某函数占用的CPU时间，包含内部调用其它函数的CPU时间|
|Excl Cpu Time|某函数占用的CPU时间，但不含内部调用其它函数所占用的CPU时间|
|**Incl Real Time**|**某函数运行的真实时间（以毫秒为单位），内含调用其它函数所占用的真实时间**|
|Excl Real Time|某函数运行的真实时间（以毫秒为单位），不含调用其它函数所占用的真实时间|
|Call+Recur Calls/Total|某函数被调用次数以及递归调用占总调用次数的百分比|
|Cpu Time/Call|某函数调用CPU时间与调用次数的比。相当于该函数平均执行时间|
|Real Time/Call|同CPU Time/Call类似，只不过统计单位换成了真实时间|

## 分析问题

这么多列，其实就是**Incl Real Time**会比较有用，表示的是包含子函数的总执行时间，通过点击children一层层往下找，知道找到你的函数为止，那就是问题所在。像我上图中也就是isInstallByPackageName函数会占用比较多的时间，然后我们在代码中找到这行代码，果不其然，是因为循环去执行了这个函数，isInstallByPackageName本身其实占用不了多少时间（但是对性能有影响是肯定的），我没测试过，暂且认为每次执行占用0.1秒，但是我这个循环实在太多了，量变引起质变，导致了程序无响应。

## 解决问题

其实很简单，就在循环体外把已安装应用列表拿出来，再循环判断是否已安装（业务逻辑，没看懂不要紧）。

写完收工。