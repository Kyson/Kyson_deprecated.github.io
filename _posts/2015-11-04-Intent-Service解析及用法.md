---
layout: post
title: Intent Service解析及用法
published: true
tags: [service,handler,thread]
---

## 简介

官网介绍：

> IntentService is a base class for Services that handle asynchronous requests (expressed as Intents) on demand. Clients send requests through startService(Intent) calls; the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.

也就是说IntentService继承自Service，用于处理异步任务，通过startService(Intent)启动该Service，并执行任务，如果有新的Intent传过来，就把任务塞进任务队列，把所有任务执行完之后会自行stopself。

## 解析

我们看下IntentService的代码，其实代码量很少，160行。

IntentService继承自Service，变量有重要的两个：

```java
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
```

ServiceHandler就是一个普通的不能再普通的Handler，不过需要注意一点，这个handler是绑定在非UI线程的Looper上的，也就是说，handleMessage是在非UI线程处理任务的。

> 这里的变量用了volatile修饰，凭我对volatile的了解还不足以解释为什么这里需要这个变量。希望有大神能为我解惑，感激不尽。

继续看代码，IntentService在oncreate的时候new了一个HandlerThread，并执行start()，也就是说创建了一个带有Looper的工作线程，然后用这个线程的Looper创建了handler，也就是刚才所说的ServiceHandler。

> 更多关于HandlerThread可以看我每日一问中2015-10-22号的问题解答[http://blog.hikyson.cn/weekly-question/](http://blog.hikyson.cn/weekly-question/)

看到这里可能还是有点迷糊，然后再看onstart方法，它是通过ServiceHandler向刚创建的HandlerThread线程塞了一个消息进去，让这个工作线程去处理。自然这个工作线程就用mServiceHandler的回调函数handlemessage去处理。

```java
private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
```

执行完之后就自动调用stopSelf(msg.arg1)停止Service。在ondestory里别忘了还会把Looper停掉。

> 如果任务还没执行完，这时候又start了一次，service会执行onStartCommand，然后继续塞一条消息进工作线程，因为此时Looper还没停掉，所以执行完上一个任务之后会拿到这个任务继续执行，直到所有任务执行完。

## 使用

### 定义

定义一个类继承IntentService，覆写构造函数和onHandleIntent(Intent intent)函数，在该函数中执行耗时的任务。

```java
public class IntentService4Test1 extends IntentService {
    private static final String TAG = "IntentService4Test1";

    public IntentService4Test1() {
        super("Just4Test");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Log.i(TAG, "任务执行完毕:" + intent.getStringExtra("tag"));
    }
}
```

启动该Service：

```java
private void start(String tag){
        Intent intent = new Intent(MainActivity.this,IntentService4Test1.class);
        intent.putExtra("tag",tag);
        startService(intent);
    }
```

测试一下：

```java
        start("first");
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                start("sec");
            }
        },2000);
```

Log打印出来的结果：

```
任务执行完毕:first
任务执行完毕:sec
```

写完收工。