
# Android性能优化--进程保活(11种方案总结)

@[TOC]

## 为什么要保活，什么是LMKD
Google 官方解释如下：
>Android Low Memory Killer Daemon (lmkd) is a process monitoring memory  
> state of a running Android system and reacting to high memory pressure  
> by killing the least essential process(es) to keep system performing  
> at acceptable levels.  
> 此处引用 [原文连接](https://android.googlesource.com/platform/system/core/+/refs/tags/android-10.0.0_r33/lmkd/) ，可以[使用工具](https://suying555.net/auth/register?code=8ogy) 访问、下载安卓源码



LMKD是一个进程，它监视正在运行的Android系统的内存状态，  
并通过杀死最不重要的进程(es)来应对高内存压力，从而使系统的性能保持在可接受的水平。   
所有应用进程都是从zygote孵化出来的，记录在AMS中mLruProcesses列表中，由AMS进行统一管理，  
AMS中会根据进程的状态更新进程对应的oom_adj值，  
这个值会传递到kernel中去，kernel有个低内存回收机制，  
在内存达到一定阀值时会触发清理oom_adj值高的进程腾出更多的内存空间。  
进程优先级可以参考 [ProcessList.java@6.0.1_r63](https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-6.0.1_r63/services/core/java/com/android/server/am/ProcessList.java)    
不同*_ADJ的有不同优先级，值越大优先级越低，  
[ProcessList.java@7.1.2_r38](https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-7.1.2_r38/services/core/java/com/android/server/am/ProcessList.java)、[ProcessList.java@android10-release](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android10-release/services/core/java/com/android/server/am/ProcessList.java)、[ProcessList.java@android-13.0.0_r52](https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-13.0.0_r52/services/core/java/com/android/server/am/ProcessList.java)  
不同版本实现稍有不同

```bash
adb shell
su    #需要root权限
# 查看minfree的值，安卓10以后没有目录/sys/module/lowmemorykiller/
cat /sys/module/lowmemorykiller/parameters/minfree
#查看adj的值
cat /sys/module/lowmemorykiller/parameters/adj
#查看进程<pid>的adj值
cat /proc/<pid>/oom_adj
cat /proc/<pid>/oom_score_adj

# android 10 以后以上方式无效，可以通过如下方式获取minfree信息，具体可参见 lmkd 源码分析
adb shell getprop sys.lmk.minfree_levels
```
内核源码参见  
[Lmkd.c@android-10.0.0_r33](https://android.googlesource.com/platform/system/core/+/refs/tags/android-10.0.0_r33/lmkd/)   
[Lmkd.cpp@android-13.0.0_r52](https://android.googlesource.com/platform/system/memory/lmkd/+/refs/tags/android-13.0.0_r52)  

解析可以参考 [Android10 lowmemorykiller](https://blog.csdn.net/zcyxiaxi/article/details/124835750)

参考 [Google Android8.0 后台限制说明](https://developer.android.google.cn/about/versions/oreo/background)

本文连接 <https://blog.csdn.net/CSqingchen/article/details/105362755>

## Service保活方案
### 1. Activity提权
原理：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素透明的 Activity，
在用户解锁时将 Activity 销毁掉，从而达到提高进程优先级的作用。  
示例代码: [Gitee: ServiceDaemon](https://gitee.com/chenjim/ServiceDaemon/tree/master/app/src/main/java/com/chenjim/daemon/activity)

### 2. Service机制(Sticky)拉活
- 将 Service 设置为 START_STICKY，利用系统机制在 Service 挂掉后自动拉活.  
- 如果service进程被kill掉，保留service的状态为开始状态，但不保留递送的intent对象。
随后系统会尝试重新创建service，由于服务状态为开始状态，
所以创建服务后一定会调用onStartCommand(Intent,int,int)方法。
如果在此期间没有任何启动命令被传递到service，那么参数Intent将为null。

- 只要 targetSdkVersion 不小于5，就默认是 START_STICKY。
但是某些ROM 系统不会拉活。并且经过测试，Service 第一次被异常杀死后很快被重启，
第二次会比第一次慢，第三次又会比前一次慢，
一旦在短时间内 Service 被杀死4-5次，则系统不再拉起。

### 3. Native拉活

详细可以参考  
博文 [Android 进程常驻（3）----native保活5.0以下方案推演过程以及代码详述](https://blog.csdn.net/marswin89/article/details/50899838)   
源码 [Github:MarsDaemon](https://github.com/Marswin/MarsDaemon)  
高版本基本不可用

### 4. “全家桶”拉活
有多个app在用户设备上安装，只要开启其中一个就可以将其他的app也拉活。  
比如手机里装了百度、百度网盘、百度钱包等等，那么打开任意一个app后，其他的app也都会被唤醒。  
**然而**我们小厂的应用很难做到一个用户手机里面安装我们的多常用个应用

### 5. 广播拉活
在发生特定系统事件时，系统会发出广播，通过在 AndroidManifest 中静态注册对应的广播监听器，
即可在发生响应事件时拉活。但是从android 7.0开始，对广播进行了限制，而且在8.0以后更加严格

目前可以注册静态广播有：
ACTION_LOCKED_BOOT_COMPLETED、ACTION_BOOT_COMPLETED、
ACTION_USB_ACCESSORY_ATTACHED、ACTION_USB_ACCESSORY_DETACHED、
ACTION_USB_DEVICE_ATTACHED、ACTION_USB_DEVICE_DETACHED、
ACTION_CONNECTION_STATE_CHANGED、ACTION_CONNECTION_STATE_CHANGED、  
ACTION_ACL_CONNECTED、ACTION_ACL_DISCONNECTED等
详细参见 [Google Android Broadcast-Exceptions](https://developer.android.google.cn/guide/components/broadcast-exceptions.html)


### 6. Service提权
创建一个前台服务用于提高app在按下home键之后的进程优先级，  
也就是使用startForeground(ID,Notification)使Service成为前台Service，  
前台服务需要在通知栏显示一条通知  
示例源码：[Gitee ServiceDaemon FrondServiceNotification.java](https://gitee.com/chenjim/ServiceDaemon/blob/master/app/src/main/java/com/chenjim/daemon/model/FrondServiceNotification.java)

**误区**：配置``<intent-filter android:priority="1000" />``提升优先级  
参考[developer@intent-filter-element](https://developer.android.com/guide/topics/manifest/intent-filter-element.html)  
**此属性对 Activity 和广播接收器都有意义**

### 7. 推送拉活
根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送。
当用户收到通知后，点击通知栏，进而拉活我们的服务。

小米消息推送[接入文档](https://dev.mi.com/console/doc/detail?pId=230)   
华为推送服务[接入文档](https://developer.huawei.com/consumer/cn/hms/huawei-pushkit)  
其它手机厂商可以搜索


### 8. JobScheduler拉活
JobScheduler允许在特定状态与特定时间间隔周期执行任务。  
可以利用它的这个特点完成保活的功能，效果即开启一个定时器，与普通定时器不同的是其调度由系统完成。

示例源码 [Gitee: ServiceDaemon@MyJobService.java](https://gitee.com/chenjim/ServiceDaemon/blob/master/app/src/main/java/com/chenjim/daemon/service/MyJobService.java)  
测试结果如下
```
2020-04-07 13:17:11.428 21328-21328/ E/MyJobService: 开启job
2020-04-07 13:17:15.685 21328-21328/ E/MyJobService: 开启job
2020-04-07 13:18:37.790 21328-21328/ E/MyJobService: 开启job
2020-04-07 13:22:07.941 21328-21328/ E/MyJobService: 开启job
2020-04-07 13:25:37.984 21328-21328/ E/MyJobService: 开启job
2020-04-07 13:29:08.087 21328-21328/ E/MyJobService: 开启job
2020-04-07 13:29:12.054 21328-21328/ E/MyJobService: 开启job
2020-04-07 13:30:08.712 21328-21328/ E/MyJobService: 开启job
```
### 9. 账户同步拉活
手机系统设置里会有“帐户”一项功能，  
任何第三方APP都可以通过此功能将数据在一定时间内同步到服务器中去。  
系统在将APP帐户同步时，会将未启动的APP进程拉活

示例源码[Github: BasicSyncAdapter ](https://github.com/googlearchive/android-BasicSyncAdapter) 或者[Gitee: ServiceDaemon ](https://gitee.com/chenjim/ServiceDaemon/tree/master/app/src/main/java/com/chenjim/daemon/account)

### 10. 双进程守护
两个进程共同运行，如果有其中一个进程被杀，  
那么另外一个进程就会将被杀的进程重新拉起  
示例源码 [Gitee: ServiceDaemon ](https://gitee.com/chenjim/ServiceDaemon/tree/master/app/src/main/java/com/chenjim/daemon/service)

### 11. 手机设置白名单、自启动、省电策略无限制等
依据不同手机，针对具体应用，引导用户设置，这个到了用户那，去设置的很少。。。


## 总结
以上方法可以提升后台服务的存活，在实际使用中很难做到100%，  
主要是因为安卓系统对后台服务的限制越来越严格

本文连接 <https://blog.csdn.net/CSqingchen/article/details/105362755>