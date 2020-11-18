
[TOC]

## 前言
- 类 `android.os.Build` [源码](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/os/Build.java)，可以[ 使用工具 ](https://suying555.net/auth/register?code=8ogy) 访问
- Android 对类 `android.os.Build ` 的介绍：  <https://developer.android.com/reference/android/os/Build>
- 在系统文件`/system/build.prop`中包含很多系统配置的key和value，可以直接使用类 [android.os.Build](https://developer.android.com/reference/android/os/Build) 提供的方法，但是不能获取所有的key对应的value，下面介绍一种方法获取其它的key对应的value

本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/51304429>


---

## 常见函数及其值
```
val build = "Build.BOARD:${Build.BOARD}\n" +
        "Build.BOOTLOADER:${Build.BOOTLOADER}\n" +
        "Build.BRAND:${Build.BRAND}\n" +
        "Build.DEVICE:${Build.DEVICE}\n" +
        "Build.DISPLAY:${Build.DISPLAY}\n" +
        "Build.FINGERPRINT:${Build.FINGERPRINT}\n" +
        "Build.HARDWARE:${Build.HARDWARE}\n" +
        "Build.HOST:${Build.HOST}\n" +
        "Build.ID:${Build.ID}\n" +
        "Build.MANUFACTURER:${Build.MANUFACTURER}\n" +
        "Build.MODEL:${Build.MODEL}\n" +
        "Build.PRODUCT:${Build.PRODUCT}\n" +
        "Build.TIME:${Build.TIME}\n" +
        "Build.TYPE:${Build.TYPE}\n" +
        "Build.USER:${Build.USER}\n" +
        "Build.Partition.PARTITION_NAME_SYSTEM:${Build.Partition.PARTITION_NAME_SYSTEM}\n" +
        "Build.VERSION.BASE_OS:${Build.VERSION.BASE_OS}\n" +
        "Build.VERSION.CODENAME:${Build.VERSION.CODENAME}\n" +
        "Build.VERSION.INCREMENTAL:${Build.VERSION.INCREMENTAL}\n" +
        "Build.VERSION.PREVIEW_SDK_INT:${Build.VERSION.PREVIEW_SDK_INT}\n" +
        "Build.VERSION.RELEASE:${Build.VERSION.RELEASE}\n" +
        "Build.VERSION.SDK_INT:${Build.VERSION.SDK_INT}\n" +
        "Build.VERSION.SECURITY_PATCH:${Build.VERSION.SECURITY_PATCH}\n"
```
结果如下：
```
Build.BOARD:sp9863a_1h10
Build.BOOTLOADER:unknown
Build.BRAND:Teclast
Build.DEVICE:p20_n6h1
Build.DISPLAY:V1.06_20210406
Build.FINGERPRINT:rockchip/rk312x/rk312x:6.0.1/MXC89K/user.fyl.20170927.202757:userdebug/test-keys
Build.HARDWARE:s9863a1h10
Build.HOST:android-build
Build.ID:QP1A.190711.020
Build.MANUFACTURER:Teclast
Build.MODEL:P20HD(N6H1)
Build.PRODUCT:p20_n6h1
Build.TIME:1617700065000
Build.TYPE:user
Build.USER:yhg229
Build.Partition.PARTITION_NAME_SYSTEM:system
Build.VERSION.BASE_OS:
Build.VERSION.CODENAME:REL
Build.VERSION.INCREMENTAL:14217
Build.VERSION.PREVIEW_SDK_INT:0
Build.VERSION.RELEASE:10
Build.VERSION.SDK_INT:29
Build.VERSION.SECURITY_PATCH:2020-04-05
```
这些信息包含设备的厂商及Android版本相关信息

---

## adb 读取 build.prop 中内容
在早期安卓版本和已经ROOT的系统，使用如下命令，可以在终端查看`/system/build.prop`中所有的key和value  
`adb shell "cat /system/build.prop"`   
新的安卓系统从安全上考虑，已经加了权限限制，无法读取，除非ROOT，如下图    
<img src="https://pic.chenjim.com/20210429162917.png-blog" width = "600"/>

有些人说可以[通过读取文件获取其中的值](https://blog.csdn.net/lzy0168/article/details/56481535)，目前也是不行的，会提示 ` Permission denied` ，原因同上。


---
## 通过反射读取 build.prop 中 key对应的value

使用以下代码可以在代码中，获取 `/system/build.prop` 中key对应的value
```
private String getBuildPropValue(String key) {
    String value = null;
    try {
        Method method = Build.class.getDeclaredMethod("getString", String.class);
        method.setAccessible(true);
        value = (String) method.invoke(new Build(), key);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return value;
}
```
使用方法如下  
`getBuildPropValue("persist.sys.timezone")`

---

## Build.getFingerprintedPartitions()

这是一个API 29（Android10）加入的API，参考自 [getFingerprintedPartitions](https://developer.android.com/reference/android/os/Build#getFingerprintedPartitions())  
得到的`List<Build.Partition>`数据类似如下
```
partition[name:bootimage,fingerprint:rockchip/rk312x/rk312x:6.0.1/MXC89K/user.fyl.20170927.202757:userdebug/test-keys,buildTimeMillis:1617700065000]
partition[name:odm,fingerprint:rockchip/rk312x/rk312x:6.0.1/MXC89K/user.fyl.20170927.202757:userdebug/test-keys,buildTimeMillis:1617699225000]
partition[name:product,fingerprint:rockchip/rk312x/rk312x:6.0.1/MXC89K/user.fyl.20170927.202757:userdebug/test-keys,buildTimeMillis:1617699225000]
partition[name:system,fingerprint:rockchip/rk312x/rk312x:6.0.1/MXC89K/user.fyl.20170927.202757:userdebug/test-keys,buildTimeMillis:1617700065000]
partition[name:vendor,fingerprint:rockchip/rk312x/rk312x:6.0.1/MXC89K/user.fyl.20170927.202757:userdebug/test-keys,buildTimeMillis:1617700065000]
```

---

##  Build.getRadioVersion()
返回的是 射频固件版本信息，结果类似如下  
`FM_BASE_18B_W20.08.3|sc9863A_modem|02-19-2020 01:21:39`

本文地址 <https://blog.csdn.net/CSqingchen/article/details/51304429>

---

## Build.getSerial()  设备标识符

这个函数是其中最复杂的一个，在API 26（Android 8.0）加入，返回硬件标识符。


- 在Android 8.0 中需要在 `AndroidManifest.xml` 添加以下权限  
  `<uses-permission android:name="android.permission.READ_PHONE_STATE"/>`  
  而且需要动态申请权限  
  `PackageManager.PERMISSION_GRANTED != activity?.checkSelfPermission(Manifest.permission.READ_PHONE_STATE))`  
  `ActivityCompat.requestPermissions(,,)`
  返回结果类似 `7c422495`，跟`adb devices`列出的设备标识符一致  
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210430103757287.png)
- 在Android 9.0 以后，持久设备标识符在附加限制之后被保护，需要权限 READ_PRIVILEGED_PHONE_STATE，
  在源码 [frameworks/core/res/AndroidManifest.xml](https://android.googlesource.com/platform/frameworks/base.git/+/refs/heads/master/core/res/AndroidManifest.xml) 中可以看到如下
  ```
  <!-- @SystemApi @TestApi Allows read access to privileged phone state.
       @hide Used internally. -->
  <permission android:name="android.permission.READ_PRIVILEGED_PHONE_STATE"
      android:protectionLevel="signature|privileged" />
  ```
  这个权限只有在privileged应用（apk内置至system/priv-app目录）才可以访问，即第三方应用无法访问。
  Android 10 开始，同样受影响的还有`TelephonyManager`的函数`getImei()、getDeviceId()、getMeid()、getSimSerialNumber()、getSubscriberId()`，参见 [文档 device-identifiers](https://source.android.com/devices/tech/config/device-identifiers)

- 总结，对于第三方应用，只有在安卓8.0设备才能获取到对应的值，其它情况要么返回 [Build#UNKNOWN](https://developer.android.com/reference/android/os/Build#UNKNOWN)，要么抛出异常`SecurityException`，要么抛出未授权异常。

- 要解决SecurityException 异常 `does not meet the requirements to access device identifiers`，就需要重新设计获取设备标识相关的代码，可以参考  
  <https://developer.android.com/training/articles/user-data-ids>

参考自 <https://developer.android.com/reference/android/os/Build#getSerial()>

---


参考自:
<http://www.cnblogs.com/onlywujun/p/3519037.html>
[java method invoke private](http://zhidao.baidu.com/link?url=CQoHbljbFb9bmfPFDjAqcYU5rnt-ggKqyFlxJZKdMz-lVXDtvRvY1WB1fGZsawhrR_HWF7JI5UU9tuFEN2Q-2K)
[ANDROID_ID(SSAID)设备标识](https://www.jianshu.com/p/076fef09c399)