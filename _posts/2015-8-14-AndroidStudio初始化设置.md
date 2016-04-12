---
layout: post
title: AndroidStudio初始化设置
---

## 代码提示

如图：

![code-completion](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/AndroidStudio%E5%88%9D%E5%A7%8B%E5%8C%96%E8%AE%BE%E7%BD%AE/code-completion.jpg)

case sensitive completion默认是首字母匹配。

> none就是不管大小写匹配，all全词匹配，firstletter第一个字母匹配。一般用none。

## 显示行号

如图：

![show-line-numbers](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/AndroidStudio%E5%88%9D%E5%A7%8B%E5%8C%96%E8%AE%BE%E7%BD%AE/show-line-numbers.jpg)

勾选 show line numbers项。

## 字体大小

### IDE的字体如图：

![theme-Colors-Fonts](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/AndroidStudio%E5%88%9D%E5%A7%8B%E5%8C%96%E8%AE%BE%E7%BD%AE/theme-Colors-Fonts.jpg)

### 代码字体如图：

![Colors-Fonts](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/AndroidStudio%E5%88%9D%E5%A7%8B%E5%8C%96%E8%AE%BE%E7%BD%AE/Colors-Fonts.jpg)

我喜欢编辑器是字号13，代码使用Source Code Pro，大小14，行距1.2

> 代码改字体需要点击save as保存自己的一套方案才能改。

## 文件编码

如图：

![File-Encodings](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/AndroidStudio%E5%88%9D%E5%A7%8B%E5%8C%96%E8%AE%BE%E7%BD%AE/File-Encodings.jpg)

eclipse中我的代码都是utf-8的，所以这边我也一直用utf-8

## 启动自动下载组件

Android Studio每次启动会自动检查sdk及组件更新，如果不翻墙的话这个过程需要很久而且会失败，可以把这个检查和下载过程去掉：

在 Android studio 安装目录的\bin子目录，找到 idea.properties 文件。在 idea.properties 文件的最后增加一行代码：

`disable.android.first.run=true`

## 代码getter和setter生成模板

如果完全按照google的代码规范，private变量以m开头，非常量static变量以s开头，那么直接使用AS的自动生成getter，setter工具会带上m和s，解决此问题只需要在

`settings -> Editor -> Code Style -> java -> Code Genetarion`

中找到Name prefix，Field的前缀设置为m，Static field前缀设置为s即可。

## 加速加载

- 可以把不需要的插件禁用，在`setting -> plugins`
- 启动时一般设置为不要打开最新的项目，而是自己选择。设置项在：
	`settings -> System Settings -> reopen last project on startup`

## 自动导包

Eclipse中我们**ctrl shift+o** 即可自动导包，Android Studio中我们需要稍微设置一下，在路径：

`Editor -> General -> Auto Import`

中勾选**Optimize imports on the fly**和**Add unambiguous imports on the fly**就可以使用相应快捷键（如果之前快捷键设置为Eclipse相同的话就只需要按**ctrl shift+o**即可）。

## Log颜色区分

默认的配色不太好看，只有error是比较清楚的，现在修改为如下配色：

|Log级别|颜色|
|---|---|
|Assert|#6D488C|
|Debug|#287CA2|
|Error|#DA0009|
|Info|#5E8000|
|Verbose|#000000|
|Warning|#D78300|

路径在：

`Settings -> Editor -> Color & Fonts -> Android Logcat`

## 插件

- Android Drawable Importer
	
	快速找图标

- FindBugs-IDEA

	findbugs不解释

- Android Parcelable Code Generator

	快速生成Parcelable

- GsonFormat

	根据gson字符串快速生成gson bean

- ButterKnife Zelezny

	根据布局快速生成ButterKnife注解	

- SelectorChapek

	根据drawable文件格式生成对应的selector（对文件命名很严格）

- CheckStyle-IDEA

	检查java代码风格插件