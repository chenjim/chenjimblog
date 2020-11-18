
[TOC]

>本文代码地址 <https://gitee.com/chenjim/QueryAppInfo>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>  

## 前言
如果应用以 Android 11（API 级别 30）或更高版本为目标平台，并查询与设备上已安装的其他应用相关的信息，则系统在默认情况下会过滤此信息。  

此过滤行为意味着您的应用无法检测设备上安装的所有应用，这有助于最大限度地减少您的应用可以访问但在执行其用例时不需要的潜在敏感信息。  
  
比如 queryIntentActivities()、getPackageInfo() 和 getInstalledApplications() 将无法获取到返回结果.   

## 问题示例
在本文示例代码中，先运行 `app2` ，在运行 `app`，从日志可以看到未获取到 `app2` 的信息  
而且还有 `PackageManager.NameNotFoundException` 异常  
![](https://pic.chenjim.com/202307311721458-blog) 


## 解决问题
如果需要能获取到 `app2` 的信息，
修改文件 `app\src\main\AndroidManifest.xml` 中 `queries` 字段如下   
```xml
    <queries>
        <package android:name="com.chenjim.app2"/>
        ......
    </queries>
```
然后，我们就可以顺利的跳转到 `app2` 了  
```kotlin
    val bt = findViewById<Button>(R.id.go_other_bt)
    if (info == null) {
        bt.isEnabled = false
    } else {
        bt.setOnClickListener {
            val cName = ComponentName(pkg, "com.chenjim.app2.App2Activity")
            val intent = Intent.makeMainActivity(cName)
            startActivity(intent)
        }
    }
```
## 问题扩展
当我们调用第三方SDK时，比如微博、微信分享，启动另外一个应用的某后台服务，均可能遇到类似的情况。  
这时，我们就需要在我们应用 `manifest` 中，用 `queries` 声明被调用者的信息，

参考文章  
<https://developer.android.com/training/package-visibility>  
<https://medium.com/androiddevelopers/package-visibility-in-android-11-cc857f221cd9> 