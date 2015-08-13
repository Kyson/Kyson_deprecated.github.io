---
layout: post
title: TT日程管理开发——适配android-5.0
---

我相信很多人对android 5.0的Material Design一见钟情，有些人比如我简直已经到了崇拜的地步，所以在查阅了大量资料之后，我决定让我的应用也用Material Design的风格润润色。

一开始我把我项目原有的theme中的自定义style（比如edittext、textview、button等等）全部注释掉，因为我们想用5.0的风格嘛，然后在theme继承v7包中的appcompat主题样式，这么一改，果然好了！发现actionbar变了，一般的控件也变了。但是有个问题，我原来的主题颜色需要替换成现在的colorprimary，所以把我自己写的布局控件等等都替换成了colorprimary对应的颜色，运行，perfect。

等等。有哪里不对。

我发现我登录页面的一个输入框的样式怎么还是4.0的？然后代码里找，原来这个控件是继承的AutoCompleteTextView，可是好奇怪，v7包难道不管这个控件么？查了一下，发现v7包只支持edittext，textview，spinner等常用的控件，AutoCompleteTextView这种控件根本没有支持，而最恶心的是，连button都不支持。。这个让我情何以堪。。以下是支持的控件：

- EditText

- Spinner

- CheckBox

- RadioButton

- SwitchCompat

- CheckedTextView

所以呢，Material Design虽好，可不是随随便便就可以做的。

接下来我就说说我是怎么把我的应用**[TT日程管理](http://zhushou.360.cn/detail/index/soft_id/2423472)**修改为meterial风格的。

> 注：本文引用了很多[http://www.jcodecraeer.com/](http://www.jcodecraeer.com/)上的文章，深怀感激。

## 效果

首先，有图有真相，看下原来的一个应用截图和现在的截图：

![TT日程管理2.4的界面](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/TT%E6%97%A5%E7%A8%8B%E7%AE%A1%E7%90%86%E5%BC%80%E5%8F%91%E2%80%94%E2%80%94%E9%80%82%E9%85%8Dandroid-5.0/tl2.0home.png)

![TT日程管理3.0.1的界面](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/TT%E6%97%A5%E7%A8%8B%E7%AE%A1%E7%90%86%E5%BC%80%E5%8F%91%E2%80%94%E2%80%94%E9%80%82%E9%85%8Dandroid-5.0/tl3.0menu.png)

你可以下载最新的版本看下(如果是第一幅图的效果，说明我还没上线3.0.1版本...)，效果还是很明显的，当然，这其中最主要的原因还是配色问题。

## 配色

material风格的配色是怎样的呢，我不擅长这方面的东西，所以给大家一个网站，在线配色用的，效果很棒：[http://www.materialpalette.com/](http://www.materialpalette.com/)，选定一个主色，一个配色，然后就可以下载自动生成的xml颜色文件了。

## 风格

接下来就是style和theme文件的编辑，因为只要给应用指定了一些主题样式的话，之后的几乎所有控件会自动着色的。

在你设置风格之前需要明白一点，android sdk提供的5.0风格的控件、theme、style只有minsdk 5.0以上的才能用的，所以我们找到了 support v7包，这个包是用来兼容5.0以下的程序的，当然，请注意这里说的是兼容，并不是表现一致！

所以第一个要做的就是把v7包作为lib导入，让我们的应用引用这个lib。（怎么引我就不说了）

之后呢，我们就可以使用类似**“Theme.AppCompat.Light.DarkActionBar”**这样的主题了（总之就是带有appcompat字样的）

我们在values下新建一个themes的文件，其中代码如下

```xml
<style name="BaseAppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="colorPrimary">@color/theme_primary</item>
    <item name="colorPrimaryDark">@color/theme_primary_dark</item>
    <item name="colorAccent">@color/theme_color_accent</item>
</style>
```

在values-v21文件夹（没有就创建）创建themes的文件，代码如下：

```xml
<style name="AppTheme" parent="@style/BaseAppTheme">
    <item name="android:navigationBarColor">@color/primary_dark</item>
</style>
```

**说明：**colorPrimary这个属性是主色，colorPrimaryDark这个是状态栏颜色，colorAccent是配色，navigationBarColor是底部导航条的颜色，看图：

![配色](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/TT%E6%97%A5%E7%A8%8B%E7%AE%A1%E7%90%86%E5%BC%80%E5%8F%91%E2%80%94%E2%80%94%E9%80%82%E9%85%8Dandroid-5.0/pattern.png)

*如果没弄错的话，刚才配色一节中那个网站下载下来的xml都有对应的颜色*

当然，你会发现一些控件并非表现得如你所愿，尤其是在低版本手机上，这时候就需要一个个调啦，比如我看到spinner的下拉列表的样式不是我要的，那我就这样，在style文件中定义个样式：

```xml
<style name="SpinnerAppTheme" parent="Widget.AppCompat.Spinner">
    <item name="android:dropDownSelector">@drawable/tl_list_item_selector</item>
</style>
```

然后在**BaseAppTheme**中添加spinner的样式：

```xml
<item name="android:spinnerStyle">@style/SpinnerAppTheme</item>
```

## 控件

就像文章一开始说的，v7包并非能让所有android原生控件着色，这时候就需要替换一些控件了，比如AutoCompleteTextView这个控件，我可以用v7包中的AppCompatAutoCompleteTextView替代。我们试下输入appcompat，代码提示有这些：

![appcompat控件](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/TT%E6%97%A5%E7%A8%8B%E7%AE%A1%E7%90%86%E5%BC%80%E5%8F%91%E2%80%94%E2%80%94%E9%80%82%E9%85%8Dandroid-5.0/appcompat_widget.png)

可以说，几乎所有无法着色的控件都可以找到替代品，简直完美~

## toolbar

5.0以前我们都用的actionbar，为了支持3.0以下我们引用了v7包，然后用的actionbaractivity（如果没有也没关系）。现在你有更好的选择，那就是**toolbar**。

想要用toolbar替换actionbar也是很方便的，首先我们要去掉actionbar。那就在theme中使用：

```
    <item name="windowActionBar">false</item>
    <item name="windowNoTitle">true</item>
```

然后在activity的布局中添加顶部的toobar。

```xml
<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="?attr/colorPrimaryDark"/>
```

注意：toobar和其他控件没什么区别，所以可以当成一个普通控件使用。但是我们把toolbar当作标题栏使用的时候需要明确告知activity我是actionbar。所以，要在activity中加上如下代码：

```java
    @Override
    protectedvoidonCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(getLayoutResource());
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        if(toolbar != null) {
            setSupportActionBar(toolbar);
        }
    }
```

还有一点，我们现在可以用AppcompatActivity替换ActionBarActivity，使用方式一样。

至此，应该说我们的应用已经拥有了material的样式了，不过总感觉少了点什么。没错，那就是动画。

## 动画

material的动画可以说是无处不在，不过我这里就说一个activity的过渡动画，首先在values-v21下的themes.xml文件中添加如下属性：

```xml
    <item name="android:windowContentTransitions">true</item>
    <item name="android:windowAllowEnterTransitionOverlap">true</item>
    <item name="android:windowAllowReturnTransitionOverlap">true</item>
    <item name="android:windowSharedElementEnterTransition">@android:transition/move</item>
    <item name="android:windowSharedElementExitTransition">@android:transition/move</item>
```

然后在activity的跳转上使用v7包的兼容方法：

```java
ActivityOptionsCompat options = ActivityOptionsCompat.makeSceneTransitionAnimation(
     activity, transitionView, DetailActivity.EXTRA_IMAGE);
ActivityCompat.startActivity(activity, newIntent(activity, DetailActivity.class),
options.toBundle());
```

写完收工！
