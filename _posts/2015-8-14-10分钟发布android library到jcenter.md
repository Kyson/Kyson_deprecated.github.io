---
layout: post
title: 10分钟发布android library到jcenter
---

首先说明一点，类似文章呢网上很多，我参考的几篇见文章结尾。

> 我使用的android studio版本是**1.3rc**，它的配套gradle版本是**2.4**的，如果你看这篇文章想要自己做的话最好和我的环境保持一致。

加入开源大家庭有些时日了，接触android studio以来一直是用的别人家的开源库，一句话就搞定了：

```groovy
dependencies {
  compile 'com.squareup.picasso:picasso:2.5.2'
}
```

真真是很炫酷啊！
分析一下，两个冒号把库路径分为了三部分：

- com.squareup.picasso，这是groupid，先记住这个名词，一会儿要用。
- picasso，artifactid，同样先记着。
- 2.5.2，版本号

**这三部分记住，很重要**

下面就以我自己的一个开源库[https://github.com/Kyson/CalendarPageView](https://github.com/Kyson/CalendarPageView)来说如何发布library到jcenter。

## 大概流程

流程图我就不画了，这边[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html)有。我们的主要工作就是本地生成aar，javadoc等文件，然后上传到bintray这个网站上，至于上传jcenter是点击一个按钮的事。

> [http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html)这篇文章有一个误区，里面提到我们要现在bintray上生成package，其实不需要哈。

## 注册bintray

这部分忽略了，看[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html)就可以了。

## 生成aar文件并上传

建立项目我就不说了，不在本文范畴内，我们主要看我们需要上传的library这个module，我的项目结构这样：

![http://git.oschina.net/cocobaby/kyson_public_log/raw/master/hikyson.cn/10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android%20library%E5%88%B0jcenter/projectpreview.png?dir=0&filepath=hikyson.cn%2F10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android+library%E5%88%B0jcenter%2Fprojectpreview.png&oid=c6fa0bae6edc68600372b87645588597612dd026&sha=86b05ca7dd77ec438325c52ef0bd6639a4d22807](http://git.oschina.net/cocobaby/kyson_public_log/raw/master/hikyson.cn/10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android%20library%E5%88%B0jcenter/projectpreview.png?dir=0&filepath=hikyson.cn%2F10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android+library%E5%88%B0jcenter%2Fprojectpreview.png&oid=c6fa0bae6edc68600372b87645588597612dd026&sha=86b05ca7dd77ec438325c52ef0bd6639a4d22807)

app这个module是测试用的，calendarpageview这个module才是我们真正的library。

### step1

首先我们要申明maven和bintray的插件依赖，在project的gradle中配置，那么现在的project gradle就是这样的：

```groovy
dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
        classpath 'com.github.dcendents:android-maven-plugin:1.2'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
```

### step2

然后我们修改calendarpageview这个module的gradle，如下：

```groovy
apply plugin: 'com.android.library'
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'
// 这个version是区分library版本的，因此当我们需要更新library时记得修改这个version
version = "1.0.0"
android {
    compileSdkVersion 22
    buildToolsVersion "23.0.0 rc2"
    resourcePrefix "CalendarPageView_"
    defaultConfig {
        minSdkVersion 9
        targetSdkVersion 21
        versionCode 1
        versionName version
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}

def siteUrl = 'https://github.com/Kyson/CalendarPageView'      // 项目的主页
def gitUrl = 'https://github.com/Kyson/CalendarPageView.git'   // Git仓库的url
group = 'com.tt' // Maven Group ID for the artifact，一般填你唯一的包名

install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                // Add your description here
                name 'a quick,simple,custom month view'  //项目描述
                url siteUrl
                // Set your license
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'kyson'       //填写开发者基本信息
                        name 'kyson'
                        email 'kyson@hikyson.cn'
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}
task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += configurations.compile
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
javadoc {
    options{
        encoding "UTF-8"
        charSet 'UTF-8'
        author true
        version true
        links "http://docs.oracle.com/javase/7/docs/api"
        title PROJ_ARTIFACTID
    }
}

artifacts {
    archives javadocJar
    archives sourcesJar
}
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"  //发布到Bintray的那个仓库里，默认账户有四个库，我们这里上传到maven库
        name = PROJ_ARTIFACTID  //发布到Bintray上的项目名字
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```

**我把几个关键点说一下。**

```groovy
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'
```

这两句话就是说我们要用maven和bintray的插件来上传东西的

`resourcePrefix "CalendarPageView_"`这个属性没搞清楚干嘛的，不过是随便填写的。
> 我猜想是上传的包里面我的资源文件都会加上这个前缀以免别人用的时候和自己的资源文件冲突。如果你有自己的理解请务必和我说哈，感激不尽。

`version = "1.0.0"`是版本号，还记得文章开始的那个依赖申明怎么写吗？三个部分组成，这就是最后一部分：版本号。

```groovy
def siteUrl = 'https://github.com/Kyson/CalendarPageView'      // 项目的主页
def gitUrl = 'https://github.com/Kyson/CalendarPageView.git'   // Git仓库的url
```

定义了两个变量，一个是项目的地址，一个是git的地址，这两地址就是用来填到bintray的项目介绍里面的，似乎不是必填项，不过我没试过可不可以不填，反正你最好先把项目上传github吧。
> 其实还有一个是issue tracker的地址，我这边没填也行的。

`group = 'com.tt'`这个是你的groupid，也就是文章开始说的依赖的三个部分的第一部分。

`repositories.mavenInstaller`中的内容照抄即可，不过

```groovy
developer {
           id 'kyson'       //填写开发者基本信息
           name 'kyson'
           email 'kyson@hikyson.cn'
}
```

这个信息填你注册bintray的信息。

再下面有几个`task`，需要注意的是

```groovy
task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += configurations.compile
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}
```

这个task把compile也包含进去了，因为有可能你的项目是有依赖其他项目的。我的没有依赖，所以这边没用到，有的话应该这么写没问题。

我们如果用的windows操作系统的话默认编码是GBK的，而如果项目中有中文注释的话可能就有问题，所以要加上这段：

```groovy
javadoc {
    options{
        encoding "UTF-8"
        charSet 'UTF-8'
        author true
        version true
        links "http://docs.oracle.com/javase/7/docs/api"
        title PROJ_ARTIFACTID
    }
}
```

其余的东西照抄即可。

这个脚本中有几个变量没有定义：

- PROJ_ARTIFACTID
- bintray.user
- bintray.apikey

PROJ_ARTIFACTID可以在project的**gradle.properties**中定义：`PROJ_ARTIFACTID=CalendarPageView`

`bintray.user和bintray.apikey`是你的bintray的用户名和apikey,apikey在哪儿找到呢？

进个人主页，点击edit进入这个页面

![http://git.oschina.net/cocobaby/kyson_public_log/raw/master/hikyson.cn/10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android%20library%E5%88%B0jcenter/apikeyplace.png?dir=0&filepath=hikyson.cn%2F10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android+library%E5%88%B0jcenter%2Fapikeyplace.png&oid=343587105556405d9cf6cd6eda4b2aeeb0feb616&sha=86b05ca7dd77ec438325c52ef0bd6639a4d22807](http://git.oschina.net/cocobaby/kyson_public_log/raw/master/hikyson.cn/10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android%20library%E5%88%B0jcenter/apikeyplace.png?dir=0&filepath=hikyson.cn%2F10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android+library%E5%88%B0jcenter%2Fapikeyplace.png&oid=343587105556405d9cf6cd6eda4b2aeeb0feb616&sha=86b05ca7dd77ec438325c52ef0bd6639a4d22807)

就能看到apikey了。

然后把`bintray.user和bintray.apikey`填写到项目的**local.properties**文件中

![http://git.oschina.net/cocobaby/kyson_public_log/raw/master/hikyson.cn/10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android%20library%E5%88%B0jcenter/localproperties.jpg?dir=0&filepath=hikyson.cn%2F10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android+library%E5%88%B0jcenter%2Flocalproperties.jpg&oid=e5b8600c81ccf65fce4109e795c34c450ccd78eb&sha=86b05ca7dd77ec438325c52ef0bd6639a4d22807](http://git.oschina.net/cocobaby/kyson_public_log/raw/master/hikyson.cn/10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android%20library%E5%88%B0jcenter/localproperties.jpg?dir=0&filepath=hikyson.cn%2F10%E5%88%86%E9%92%9F%E5%8F%91%E5%B8%83android+library%E5%88%B0jcenter%2Flocalproperties.jpg&oid=e5b8600c81ccf65fce4109e795c34c450ccd78eb&sha=86b05ca7dd77ec438325c52ef0bd6639a4d22807)

> 一定要填写这个文件中！而且检查一下**.gitignore**文件是否有把**local.properties**忽略提交。因为apikey是不能被别人看到的。

到这里所有准备工作都做完了，接下来我们试下能不能生成上传文件，运行

`> gradlew install`

打印出**BUILD SUCCESSFUL**，这时候我们build文件夹下应该有aar等文件了，然后执行

`> gradlew bintrayUpload`

打印出**SUCCESSFUL**就ok了。

检查是否上传OK就看[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html)这篇文章的**第五部分：把library上传到你的bintray空间**，我这边不赘述了。

## 添加库到jcenter

上面做的所有事情只不过是把我们的库放到了我们自己的bintray中，要想让大家都能用jcenter中的库，我们只需要点击bintray上我的项目中的**add to jcenter**即可

> 会让你填comment，随便填点东西就可以等他审核，通过会发邮件给你的。

成功之后我们去网站上看下[http://jcenter.bintray.com/com/tt/calendarpageview/1.0.0/](http://jcenter.bintray.com/com/tt/calendarpageview/1.0.0/)，其实路径就是groupid+artifactid+version组成的。

## 使用

可以新建一个module试一下(groupid : artifactid : version):

```groovy
dependencies {
    compile 'com.tt:calendarpageview:1.0.0'
}
```

## 问题

一般来说你在执行`> gradlew install`和`> gradlew bintrayUpload`命令的时候会遇到很多错误，我这边就简单说下我遇到的一个问题。

如果告诉你`"org/gradle/api/publication/maven/internal/DefaultMavenFactory"`也不说什么原因，有可能是gradle版本问题，默认的是2.4，我换了2.3就可以了。

只要在**gradle/wrapper/gradle-wrapper.properties**中把**distributionUrl**换成**https://services.gradle.org/distributions/gradle-2.3-all.zip**，重新打开android studio。

其他的错误看情况而定，我没遇到过，必要的时候可以打印日志看下具体错误。

### 参考文章
- [http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html)
- [http://blog.csdn.net/maosidiaoxian/article/details/43148643](http://blog.csdn.net/maosidiaoxian/article/details/43148643)