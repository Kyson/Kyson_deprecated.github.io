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