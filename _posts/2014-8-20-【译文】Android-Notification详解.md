---
layout: post
title: Android-Notification详解
---

原文：[http://developer.android.com/guide/topics/ui/notifiers/notifications.html](http://developer.android.com/guide/topics/ui/notifiers/notifications.html)

本文用V4包里的`NotificationCompat.Builder`构建通知，因为`Notification.Builder`这个类是在3.0以后才有的，如果你的应用最低支持3.0的话也可以用`Notification.Builder`来构建通知。

## 设计方面的考虑

因为android已经出了5.0版本了，这个版本和以前有些不太一样，包括通知，所以你还是要先看看5.0的通知设计。这边原文没有展开讲，我讲几个注意点：

- 外观上的变化，这个看一下就ok
- 锁屏的通知，这个应该是比较有用的，你可以看到锁屏的时候QQ等应用会在上面显示消息。你也可以忽略这个。
- 有一个高优先级的通知：heads-up notifications，这个通知其实就是在屏幕顶部显示一个通知，而不是在通知栏里面。
- 还有一个云同步的通知，就是你在这个设备上消除了这个通知，你的其他设备也会同步消除掉的。

## 创建通知

创建分三步，把UI、Action等信息给`NotificationCompat.Builder`对象，调用`NotificationCompat.Builder.build()`创建出`Notification`，用`NotificationManager.notify()`进行通知。

## Notification必要元素

一个通知必须包括以下一个元素：

- small icon，对应API是`setSmallIcon()`
- 标题，对应API是`setContentTitle()`
- 内容，对应API是`setContentText()`

## Notification可选元素

原文没有直接给出，可选的元素很多，具体参考API文档吧:
[http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)

## Notification执行的动作

虽然说这个action是可选的，不过你最好加上至少一个，比如点击跳转到你指定的一个页面之类的，这个页面也可以提示用户更多他想了解的东西。

你可以给通知指定很多动作，反正点击通知的动作最好都加上，因为这个最常用。还有就是4.1版本加上了一个功能，就是通知可以显示按钮，这个功能需要注意的东西还是有点麻烦的，因为4.1才会提供附加的按钮，那么4.1之前的呢，你要保证你的应用能给所有版本的设备都提供一样的功能，怎么做？Google给出了一个解决方案：

> 比如你想要在通知栏附加两个按钮，开始和停止，而4.1之前的版本不会有这两个按钮，这时候你需要有一个Activity，这个activity会提供这个两个方法，4.1之前的设备点击通知栏会跳转到这个按钮，用户在Activity中可以执行开始和停止操作，4.1以后的设备用户可以在Activity中执行开始和停止操作，也可以直接在通知栏上执行这两个操作。

怎么指定动作，使用PendingIntent，然后配合不同的API处理手势，比如setContentIntent()指的是点击动作，然后给它指定一个PendingIntent来执行想要的操作即可。

*这块自我感觉没翻译好，Talk is cheap,看demo吧*

## 通知优先级

用`NotificationCompat.Builder.setPriority()`指定优先级，但是不是说所有的通知都是优先级越高越好的，一般来说是这样：

DEFAULT, HIGH, 和 MAX的优先级是会打断用户的，它可能是和其他人有关系的事情，可能是有时效性的事情，也有可能是一旦通知到用户，用户就会立刻改变自己行程的事情，反之，你就应该用低优先级的通知（LOW 和 MIN）

## 创建简单的通知

```java
NotificationCompat.Builder mBuilder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!");
// Creates an explicit intent for an Activity in your app
Intent resultIntent = new Intent(this, ResultActivity.class);

// The stack builder object will contain an artificial back stack for the
// started Activity.
// This ensures that navigating backward from the Activity leads out of
// your application to the Home screen.
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
// Adds the back stack for the Intent (but not the Intent itself)
stackBuilder.addParentStack(ResultActivity.class);
// Adds the Intent that starts the Activity to the top of the stack
stackBuilder.addNextIntent(resultIntent);
PendingIntent resultPendingIntent =
        stackBuilder.getPendingIntent(
            0,
            PendingIntent.FLAG_UPDATE_CURRENT
        );
mBuilder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// mId allows you to update the notification later on.
mNotificationManager.notify(mId, mBuilder.build());
```
用build指定一些基本信息，`mBuilder.setContentIntent(resultPendingIntent);`指定点击事件，`mNotificationManager.notify(mId, mBuilder.build());`进行通知。

到这里一个通知其实就OK了，剩余的不翻译了。