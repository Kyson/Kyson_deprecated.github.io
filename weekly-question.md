---
layout: page
title: 每周一问
permalink: /weekly-question/
---

*每周一个问题，欢迎提问及回答（问题主要为Android、java及周边）*

#### 2015-12-03

**Q:如何查找导致ANR的原因**

**A：**总的来说，ANR还是由于APP在系统指定的时间间隔内（一般来说是5秒）没有响应，即主线程被阻塞了，主线程的消息队列在这段时间内没有来得及处理下一个消息。
那么导致ANR的原因自然我们就知道是主线程执行了某些耗时操作（其实有些操作也可以认为是耗时，也可以认为是不耗时的，量变引起质变，量大了也就可以认为是耗时操作了），一旦我们的代码中有耗时操作，怎么去找到它？就看下这篇文章：[Android-TraceView](http://blog.hikyson.cn/Android-TraceView%E7%9A%84%E4%BD%BF%E7%94%A8/)

---

#### 2015-11-03

**Q:IntentService的原理及使用方法。**

**A：**直接看文章吧：[IntentService解析及用法](http://blog.hikyson.cn/Intent-Service%E8%A7%A3%E6%9E%90%E5%8F%8A%E7%94%A8%E6%B3%95/)

---

#### 2015-10-22

**Q:Android主线程是怎么回事？怎么创建自己的“主线程”？**

**A：**众所周知，Android应用运行在所谓的UI进程中，为了更好的用户体验，耗时操作必须放在子线程，而UI更新必须在UI线程也就是主线程中。

谈到线程就必须说到handler、looper及message queue，这几个概念这边就不多说了，网上很多，android的主线程和其他线程一样的处理流程：UI更新操作通过handler发送到主线程的队列里，然后主线程的looper不停地从这个队列里拿出来消息（这边就是更新UI），然后做出相应的更新动作。

既然主线程和其他线程没区别，那我们可以自己来创建一个“主线程”。

不用重复造轮子，android有一个类叫做HandlerThread，这个Thread自带Looper，免去了我们要自己去创建Looper的步骤。使用起来也比较便利：

```java
mHandlerThread = new HandlerThread("handler_thread");
mHandlerThread.start();
mMyHandler = new MyHandler(mHandlerThread.getLooper());
mMyHandler.sendEmptyMessage(1);
```

创建一个HandlerThread实例，启动，这时候这个线程中有一个消息队列处于阻塞状态，Looper不停地从这个队列中取消息（没有消息就阻塞）。这时候我们的HandlerThread线程就相当于主线程，只不过UI线程是用来更新UI，而我们这个还没有定义自己的处理流程。

其实我们可以进去看下HandlerThread类，代码也很简单，run方法中调用Looper.prepare();所谓prepare就是给当前线程创建一个队列和Looper。然后调用Looper.loop()，这个方法是一个死循环，就是从队列拿消息处理。

*这个问题我解释起来感觉比较晦涩，已经是基于对handler、Looper有一定了解的基础上做的回答。*

---

#### 2015-10-12

**Q:为什么说TCP是面向连接的协议，而UDP是面向无连接的协议？**

**A：**TCP在进行真正的数据通信之前会有三次握手，第一次发送端A发送一个SYC请求，接收端B接收到之后应答一个ACK包，A拿到ACK包之后再发送一个应答B的ACK包的ACK包。

个人觉得如果是单纯想建立连接的话两次就够了（有这个想法很大程度上是因为我还是个网络菜鸟），因为B接收到A的SYC时候，B就可以认为线路是通的，而A接收到B的ACK之后，A就可以认为线路是通的，这样A和B其实就可以通讯了。

但是，在这个握手过程中不仅仅希望达成一个能够通讯的目的，而是还会有其他用途，比如：通讯的数据包长度多大。

比如A觉得需要以12个字节为一个数据包长度，然后B返回说不行，还需要再小一点，就8吧，A收到了之后如果没再给B发送确认相应的话，B就不知道A是不是同意这个要求，所以需要三次握手。

这个仅仅是个人想法，希望有大神能给我一个正确的解惑。

TCP的三次握手结束之后相当于A和B之间有一个连接，之后发送的任何消息只要通过这个连接就可以了。

而UDP不需要握手，A想发什么数据直接发送，至于B能不能收到根本不在乎。所以我们说TCP是面向连接，而UDP无连接。

更多关于TCP/IP的概念看我的一篇博文：[TCP/IP概要](http://blog.hikyson.cn/TCP-IP%E6%A6%82%E8%A6%81/)


<br/>
  
<hr/>

{% include disqus.html %}
