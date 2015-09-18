---
layout: post
title: Android-M动态权限获取
tags: [Android-M,权限,permission]
---

> 本文原作者：

Android M引入了新的应用权限模型，目前官方已有中文版对其特性进行了概述：[http://developer.android.com/intl/zh-cn/preview/features/runtime-permissions.html](http://developer.android.com/intl/zh-cn/preview/features/runtime-permissions.html)

本文结合官方的文档以及一些国外的博文加上近期测试的遇到的一些问题，提取出其中的关键点，整理出相对更简明的文档供大家以后在做适配6.0的时候参考。

## 概述

新的权限获取方式除了要求像之前版本一样在AndroidManifest文件中静态申请之外，应用还需根据需要请求权限，方式采用向用户显示一个请求权限的对话框。这些被动态申请的权限可以在系统设置中被手动关闭。另外，对于类别为NORMAL的权限，仍然只需要在AndroidManifest文件中静态申请，系统安装时会直接获取，对于NORMAL权限下文有详细的说明。

![permission-normal](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-M%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E8%8E%B7%E5%8F%96/permission-normal.png)

## NORMAL权限

以下权限称为NORMAL权限，在应用安装的时候就会被自动获取而且不能被用户手动取消。

```java
android.permission.ACCESS_LOCATION_EXTRA_COMMANDS
android.permission.ACCESS_NETWORK_STATE
android.permission.ACCESS_NOTIFICATION_POLICY
android.permission.ACCESS_WIFI_STATE
android.permission.ACCESS_WIMAX_STATE
android.permission.BLUETOOTH
android.permission.BLUETOOTH_ADMIN
android.permission.BROADCAST_STICKY
android.permission.CHANGE_NETWORK_STATE
android.permission.CHANGE_WIFI_MULTICAST_STATE
android.permission.CHANGE_WIFI_STATE
android.permission.CHANGE_WIMAX_STATE
android.permission.DISABLE_KEYGUARD
android.permission.EXPAND_STATUS_BAR
android.permission.FLASHLIGHT
android.permission.GET_ACCOUNTS
android.permission.GET_PACKAGE_SIZE
android.permission.INTERNET
android.permission.KILL_BACKGROUND_PROCESSES
android.permission.MODIFY_AUDIO_SETTINGS
android.permission.NFC
android.permission.READ_SYNC_SETTINGS
android.permission.READ_SYNC_STATS
android.permission.RECEIVE_BOOT_COMPLETED
android.permission.REORDER_TASKS
android.permission.REQUEST_INSTALL_PACKAGES
android.permission.SET_TIME_ZONE
android.permission.SET_WALLPAPER
android.permission.SET_WALLPAPER_HINTS
android.permission.SUBSCRIBED_FEEDS_READ
android.permission.TRANSMIT_IR
android.permission.USE_FINGERPRINT
android.permission.VIBRATE
android.permission.WAKE_LOCK
android.permission.WRITE_SYNC_SETTINGS
com.android.alarm.permission.SET_ALARM
com.android.launcher.permission.INSTALL_SHORTCUT
com.android.launcher.permission.UNINSTALL_SHORTCUT
```

## 权限组

新的权限模型中还提出了一个权限组的概念，可以简单理解为**如果一个权限组内的某个权限被获取了，那么这个组中剩余的权限也会被自动获取**。例如：`android.permission-group.CALENDAR`中的`android.permission.WRITE_CALENDAR`
权限被获取，那么应用会自动获取`android.permission.READ_CALENDAR`权限。

![permission-group](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-M%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E8%8E%B7%E5%8F%96/permission-group.png)

## 一般动态获取方法

### tip1

判定是否有权限：checkSelfPermission()

### tip2

如果没有权限，弹出dialog给用户选择：requestPermission()，第二个参数code与onRequestPermissionResult()方法中的code对应

```java
if(checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
    requestPermissions(
            new String[] { Manifest.permission.WRITE_EXTERNAL_STORAGE },
            REQUEST_CODE_ASK_PERMISSON);
}
```

### tip3

判断用户是否确认了权限onRequestPermissionResult ()

```java
public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {
    switch (requestCode) {
    case REQUEST_CODE_ASK_PERMISSON:
        if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            // Permission Granted
        } else {
            // Permission Denied
        }
        break;
    default:
        super.onRequestPermissionsResult(requestCode, permissions,
                grantResults);
    }
}
```

### tip4

在弹出权限选择的对话框前给用户show一个dialog，用于引导用户进行选择。shouldShowRequestPermissionRationale()

```
if (checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
    if (!shouldShowRequestPermissionRationale(Manifest.permission.WRITE_EXTERNAL_STORAGE)) {
        // Explain to the user why we need to read the contacts
        showMessage("请允许应用对SD卡进行读写操作",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog,
                            int which) {
                        requestPermissions(
                                new String[] { Manifest.permission.WRITE_EXTERNAL_STORAGE },
                                REQUEST_CODE_ASK_PERMISSIONS);
                    }
                });
        return;
    }
    requestPermissions(
            new String[] { Manifest.permission.WRITE_EXTERNAL_STORAGE },
            REQUEST_CODE_ASK_PERMISSIONS);
}

private void showMessage(String message,
        DialogInterface.OnClickListener okListener) {
    new AlertDialog.Builder(MainActivity.this).setMessage(message)
            .setPositiveButton("OK", okListener).create().show();
}
```

![permission-ask-1](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-M%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E8%8E%B7%E5%8F%96/permission-ask-1.png)

![permission-ask-2](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-M%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E8%8E%B7%E5%8F%96/permission-ask-2.png)

### tip5

如果用户在选择权限对话框拒绝了某个权限的申请，那么再次申请该权限时会多出一个“不再询问”的checkbox，如果勾选，那么即便程序再调用requestPermission()，对话框也不会再弹出了。

![permission-deny-ever](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-M%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E8%8E%B7%E5%8F%96/permission-deny-ever.png)

### tip6

一次性处理多个权限申请

```java
private void requestPermissions() {
    List<String> permissionsNeeded = new ArrayList<String>();

    final List<String> permissionsList = new ArrayList<String>();
    if (!addPermission(permissionsList, Manifest.permission.READ_PHONE_STATE))
        permissionsNeeded.add("Phone State");
    if (!addPermission(permissionsList, Manifest.permission.WRITE_EXTERNAL_STORAGE))
        permissionsNeeded.add("Write SDcard");
    if (!addPermission(permissionsList, Manifest.permission.WRITE_CONTACTS))
        permissionsNeeded.add("Write Contacts");

    if (permissionsList.size() > 0) {
        if (permissionsNeeded.size() > 0) {
            // Need Rationale
            String message = "You need to grant access to " + permissionsNeeded.get(0);
            for (int i = 1; i < permissionsNeeded.size(); i++)
                message = message + ", " + permissionsNeeded.get(i);
            showMessageOKCancel(message,
                    new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            requestPermissions(permissionsList.toArray(new String[permissionsList.size()]),
                                    REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS);
                        }
                    });
            return;
        }
        requestPermissions(permissionsList.toArray(new String[permissionsList.size()]),
                REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS);
        return;
    }
}
private boolean addPermission(List<String> permissionsList, String permission) {
    if (checkSelfPermission(permission) != PackageManager.PERMISSION_GRANTED) {
        permissionsList.add(permission);
        if (!shouldShowRequestPermissionRationale(permission)){
            return false;
        }
    }
    return true;
}
```

## 兼容

### tip1

如果M(targetSDK<23)之前的应用的跑在系统为6.0的机器上，还是会在安装的时候统一申请权限。但是用户在系统设置里面还是可以手动关掉权限的开关，这时会弹出提示：

![permission-compat-tip](https://raw.githubusercontent.com/Kyson/Kyson.github.io/master/images/post_img/Android-M%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E8%8E%B7%E5%8F%96/permission-compat-tip.png)

Google官方给出的解释是：**即便用户关闭了权限，该操作也不一定会导致异常，程序也不会崩溃，返回值为0或者NULL**。

### tip2

如果targetSDK=23的应用跑在了6.0版本之前的系统上(如5.1.1)，且应用中采取动态的方式获取了权限，那么程序会崩溃，需要做适当的适配。

**方式1：**

```java
if (Build.VERSION.SDK_INT >= 23){
    // Anroid M
}else {
    // pre-Android M
}
```

**方式2：**

导入最新版本的Support-v4包，最新版本为23.0.1

- checkSelfPermission()用ContextCompat.checkSelfPermission()代替
无论应用是跑在M上还是之前的版本，如果权限被获取方法返回PERMISSION_GRANT,如果不是返回PERMISSION_DENIED。
- requestPermissions()用ActivityCompat.requestPermissions()代替
如果在M之前的版本使用，OnRequestPermissionsResultCallback 方法会立即返回权限获取的结果PERMISSION_GRANT或PERMISSION_DENIED，而不会弹出对话框让用户选择。
- shouldShowRequestPermissionRationale()用ActivityCompat.shouldShowRequestPermissionRationale()代替
如果在M之前的版本使用，返回false.
- 如果要在Fragment内使用，需要导入Support-v13包，最新版本23.0.1。
FragmentCompat.requestPermissions()
FragmentCompat.shouldShowRequestPermissionRationale()

## 存在的问题

目前测试发现动态获取SD卡写权限后，直接操作SD卡，仍旧会报未获取到权限异常：

`DownThread0-->open failed:EACCES(Permission denied)`

只有重启应用后才能真正获取到权限。Stack Overflow上有人回答可能是因为6.0preview版本的问题。根本原因未能定位，但此问题真实存在。