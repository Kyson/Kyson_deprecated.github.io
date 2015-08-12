---
layout: post
title: Android中AIDL使用
---

> 可以在[http://git.oschina.net/cocobaby/AidlTryer](http://git.oschina.net/cocobaby/AidlTryer)下载demo。

## AIDL概述

AIDL和其他的接口定义语言（IDL）工作原理类似。它允许你定义一些接口，通过跨进程通信原理（IPC），
使得客户端和服务端进行通信。
在android系统中，我们知道，每个应用运行在独立的进程中。为了在进程之间通信，我们必须把一些对象分解成系统能够理解的原子对象，然后按一定形式编组，跨过进程的边界。

> 注意：你只有在跨进程通信的时候才需要使用AIDL

在看这篇文章之前，你应该先了解bound service（就是一般我们以bind形式使用的service）。

## 使用步骤

我们以我自己写的demo为例来说明

### 建立一个服务端

#### step1

创建一个.aidl的文件

语法和java类似，只是要注意它并不是支持所有java类型。下面是支持的类型：

- 所有基础类型
- String
- CharSequence
- List（list中的类型必须也是要支持的！）
- Map（map中的类型必须也是要支持的！）
- android 序列化的类型（parcelable）（不知道为什么，我没看到android官网有写这个，没试过，不过应该可以）

```java
package com.tt.aidltryer.serviceinterface;

/**
 * aidl接口 <功能简述> <Br>
 * <功能详细描述> <Br>
 * 
 * @author Kyson
 */
interface IReplyer {
    //客户端发起提问
    void qustion(String qustion);
    //回答
    String answer();
}
```

> 注意：package 和 import（如果有的话）的代码必须要写，你可以先写一个java的普通接口，然后复制粘贴到这个文件里来。

如果上面的aidl文件没有出错的话，gen文件夹下会相应生成一个同名的java文件（`IReplyer.java`）

定义的这个aidl是干嘛的呢？
我的理解就是它相当于android提供给你的一个工具，你只要提供这个接口，android就会自动给你生成一个相当复杂的java文件（就是上面讲的这个文件），这样你就可以省下不少事情。
而你定义的这个接口就定义了服务器和客户端通信的一个“协议”，通过这些“协议”就能传递数据咯（其实就是java接口的一般用途）。

> 我这边定义的接口就是客户端发送String类型的qustion过来，服务器发送String类型的answer回去。

#### step2

创建一个普通的service

```java
public class ReplyerService extends Service {

    private Stub replyStub = new Stub() {

        private String mQustion;

        @Override
        public void qustion(String qustion) throws RemoteException {
            mQustion = qustion;
        }

        @Override
        public String answer() throws RemoteException {
            if ("h r u?".equals(mQustion)) {
                return "i am fine,and u?";
            }
            return "all right , nothing here.";
        }
    };

    @Override
    public IBinder onBind(Intent intent) {
        Log.i("kyson", "服务端已经绑定...");
        return replyStub;
    }

    @Override
    public boolean onUnbind(Intent intent) {
        Log.i("kyson", "服务端已经解绑，等待下一次绑定...");
        return true;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.i("kyson", "服务端正在运行...");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.i("kyson", "服务端已经销毁...");
    }
}
```

看到`onBind`方法中返回了一个IBinder对象，这个对象必须是刚才那个自动生成的`IReplyer.java`中定义的抽象stub类的对象，而你需要实现的方法就是刚才在aidl文件中定义的接口。

然后，在manifest中声明这个service，这个别忘了，否则会运行报错。

```xml
    <service
            android:name="com.tt.aidltryer.services.ReplyerService"
            android:process=":remote" >
            <intent-filter>
                <action android:name="android.intent.action.ReplyerService" />

                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </service>
```

到此服务端的就编写完成了，接下来我需要写一个运行在不用进程的客户端，来与服务端通信。

### 建立客户端

#### step1

首先要把服务端的aidl文件拷贝到客户端来

> 注意：包名必须一样！比如服务端刚才的路径是`com.tt.aidltryer.serviceinterface.IReplyer`，那么客户端必须也放到相同的包下

#### step2

和普通的bound service一样使用，提供一个`ServiceConnection`，代码如下：

```java
private ServiceConnection conn = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.i("kyson", "客户端已经连上服务器");
            mIReplyer = Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.i("kyson", "客户端断开与服务器连接");
        }
    };
```

在连接到服务端的时候返回一个`IReplyer`，这个接口可以

绑定service：

```java
    Intent intent = new Intent("android.intent.action.ReplyerService");
    bindService(intent, conn, Context.BIND_AUTO_CREATE);
```

#### step3

和服务端交互

通过`mIReplyer`对象，我们传递一些数据，像这样：

```java
findViewById(R.id.textView).setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                try {
                    String ques = "hey!";
                    Log.i("kyson", "发送：" + ques);
                    mIReplyer.qustion(ques);

                    String ans = mIReplyer.answer();
                    Log.i("kyson", "回答：" + ans);
                    mAnswer.setText(ans);

                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
```

点击按钮，发送"hey！"，然后接收服务器返回的回应。

OK，写完收工！大家可以下载demo看看。