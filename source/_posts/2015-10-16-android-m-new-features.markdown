---
layout: post
title: "Android M 新特性"
date: 2015-10-16 14:52:54 +0800
comments: true
categories: 
---

### 指纹识别API

调用FingerprintManager里面的authenticate()方法就可以很轻松的进行指纹识别，当然，我们的app需要
实现一个UI来提醒用户进行指纹识别，

使用指纹识别还需要加入权限

```java
<uses-permission android:name="android.permission.USE_FINGERPRINT" />
```

如果使用模拟器来进行测试的话，可以通过adb发送一个指纹来完成测试的

```java
adb -e emu finger touch <finger_id>
```

### 删除了Apache HTTP Client

### 用BoringSSL代替OpenSSL

在使用NDK的时候，不能再像以前一样使用libcrypto.so或libssl.so这些库了

### 硬件标识

WifiInfo.getMacAddress() 和 BluetoothAdapter.getAddress()已经不能再用了，Android M 对这两个方法
的返回只会是一个固定的值 02:00:00:00:00:00

而且，如果我们想获取附近的WiFi列表或蓝牙列表，Android M 之后就要加上权限了

```java
ACCESS_FINE_LOCATION
ACCESS_COARSE_LOCATION
```

我们不能再像以前一样随意的删除或修改WifiConfiguration了，现在只能够删除或修改自己创建的WifiConfiguration


### Doze和App Standby

Doze：不充电，屏幕关闭，静止不动一段时间后，就会进入这个模式，它会让系统处于睡眠的状态，但是它
还是会周期性的唤醒一小段时间，这样子，APP就可以进行一些同步操作以及系统就会处理一些延迟的操作

App Standby：进入这个模式后，系统就会把长时间不使用的APP判断为暂停使用的了，如果没有
进行充电的话，那么系统就会把这些APP的网络关闭，一些同步的操作都会暂停

只要系统是Android M 以上的，这两种模式就会生效，无论你APP的target API指定的是几

进行Doze之后，APP将无法进行网络访问，无法wake locks,AlarmManager (setExact(), setWindow())这些都
会等待到周期性唤醒的那一小段时间来处理，如果需要alarm起作用，那么就需要使用setAndAllowWhileIdle()
或setExactAndAllowWhileIdle() （这两个方法只能唤醒15分钟），需要注意的是，setAlarmClock()是照常起作用的，系统会提前一点时间
退出Doze模式来让它起作用
在这个模式下，无法搜索wifi，无法sync adapter, 无法调度JobScheduler

对于推送类的应用，Google推荐用GCM（Google Cloud Message)


对于App Standby：下面的三种情况是不会进入这个模式的

* 当用户显式得运行了APP

* APP有一个foreground的进程（包括一个activity或者一个foreground service 或者被另外的一个activity
或foreground service使用

* APP发送一个Notification，被用户在锁屏界面看到了或通知栏看到了

要注意的是，如果设备长时间处于闲置状态，那么系统就会允许这些APP访问网络1次每天


## 白名单

Android对于上面那两种模式设置了一个白名单，白名单里面的APP可以进行网络访问和partial wake locks
但其他限制依然起作用

isIgnoringBatteryOptimizations()可以判断出自己是不是在白名单里面

添加白名单：Settings > Battery > Battery Optimization

REQUEST_IGNORE_BATTERY_OPTIMIZATIONS  权限申请

ACTION_IGNORE_BATTERY_OPTIMIZATION_SETTINGS  跳转到settings

ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS 弹框申请


## adb测试


```java
$ adb shell dumpsys battery unplug
$ adb shell dumpsys deviceidle step
$ adb shell dumpsys deviceidle -h
```


```java
$ adb shell dumpsys battery unplug
$ adb shell am set-inactive <packageName> true</packageName>
```

```java
$ adb shell am set-inactive <packageName> false
$ adb shell am get-inactive <packageName></packageName>
```

