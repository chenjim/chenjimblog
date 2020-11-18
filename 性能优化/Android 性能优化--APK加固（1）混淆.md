
[TOC]

> 本文首发地址：<https://h89.cn/archives/211.html>  
> 最新更新地址：<https://gitee.com/chenjim/chenjimblog>


## 为什么要开启混淆
先上一个 [简单示例 MainActivity.kt ](https://gitee.com/chenjim/ProguardDemo)  
```kotlin
class MainActivity : AppCompatActivity() {
    private val p = Person("chenjim", 18)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Log.d("MainActivity", p.toString())
    }
}
```
未开启混淆编译的APK[逆向反编译](https://github.com/skylot/jadx)结果如下，能够清晰看到源码实现：  
![](https://pic.chenjim.com/202402252035340.png-blog)  
开启混淆编译的APK[逆向反编译](https://github.com/skylot/jadx)结果： 
![](https://pic.chenjim.com/202402252037681.png-blog)  
通过对比，开启混淆后，很难理解反编译结果的含义，进而可以减少抄袭、盗版，保护原著。  
此外，还可以移除无用的资源文件、类、方法，缩短类和成员名称，进而避免64K引用显示、减少APK的体积。


## 如何开启代码混淆
只需要修改 `build.gradle.kts` 中 isMinifyEnabled 为 true 即可  
```kotlin DSL
// app 目录 build.gradle.kts
buildTypes {
    release {
        isMinifyEnabled = true  

        isShrinkResources = true

        // 配置文件 
        proguardFiles(
            getDefaultProguardFile("proguard-android-optimize.txt"),
            "proguard-rules.pro"
        )
    }
}
```
只有在 Release 版本即 `gradlew assembleRelease` 时生效  


## 如何开启资源压缩
默认并不会开启资源压缩，可以添加 `isShrinkResources = true` 开启。  
开启后会自动移除未使用的资源文件，部分资源文件会进行进一步压缩大小。  
（使用 webp 格式资源，可以进一步减小APP体。）  
如果 isMinifyEnabled 未开启，是无法开启 isShrinkResources .    
如果想保留未使用的资源在 release.apk ，可以添加 `app/src/main/res/raw/keep.xml`，通过如下格式保留。  
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@drawable/test_skin,@layout/l_used_a" />
```

## 代码混淆配置  
如上配置中，即使 proguard-rules.pro 无内容， getDefaultProguardFile 会获取使用默认配置文件 `proguard-android-optimize.txt`。  
我们可以添加如下命令，打印出配置文件的路径。  
 `println("proguard-path:${getDefaultProguardFile("proguard-android-optimize.txt")}")` 
相应的配置文件在 gradle 编译 assembleRelease 时生成。  
内容参见 [app/build/intermediates/default_proguard_files/global/proguard-android-optimize.txt-*](https://gitee.com/chenjim/ProguardDemo/blob/main/default_proguard_files/global/proguard-android-optimize.txt-8.2.2)   
我们可以从中了解一些常见的配置使用方法，部分用法的含义如下   
```bash
# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
-optimizationpasses 5

# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames

# 指定不去忽略非公共库的类
-dontskipnonpubliclibraryclasses
```
```bash
# 这句话能够使我们的项目混淆后产生映射文件
# 包含有类名->混淆后类名的映射关系
-verbose

# 指定不去忽略非公共库的类成员
-dontskipnonpubliclibraryclassmembers
```
```bash
# 不做预校验，preverify是proguard的四个步骤之一，Android不需要preverify，去掉这一步能够加快混淆速度。
-dontpreverify

# 保留Annotation不混淆
-keepattributes *Annotation*,InnerClasses
```
```bash
# 避免混淆泛型
-keepattributes Signature

# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

# 指定混淆是采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/cast,!field/*,!class/merging/*
```
```bash
# 保留指定的类 
-keep public class com.google.vending.licensing.ILicensingService

# 保留继承自 View 类的 set 和 get .
-keepclassmembers public class * extends android.view.View {
    void set*(***);
    *** get*();
}

# 保留 Javascript 接口.
-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}
```
可以按照如上规则在 proguard-rules.pro 添加项目需要的特殊配置。  
还可以在代码中添加注解 `@Keep` 避免被混淆。
```kotlin
data class PersonKeep(
    @Keep
    val name: String, 
    val age: Int
)
```
如果引用的 AAR 包含混淆配置，我们的 APP 中可以不用单独配置。  
最终的混淆配置文件在如下文件中   
`app/build/outputs/mapping/release/configuration.txt`   
最终把什么混淆为什么，可以在如下文件中知晓，下一小节Crash分析也会用到。  
`app/build/outputs/mapping/release/mapping.txt`  
建议发布 Release APK 时，同时附上目录 `app/build/outputs/mapping/release/` 中所有输出文件。  

## 代码混淆后，Crash 问题定位
一般情况，我们拿到错误日志（可以使用 `adb logcat > logcat.log` 导出）如下，
```log
02-25 23:26:47.482 11040 11094 E AndroidRuntime: Process: com.chenjim.proguard, PID: 11040
02-25 23:26:47.482 11040 11094 E AndroidRuntime: java.lang.RuntimeException: test crash
02-25 23:26:47.482 11040 11094 E AndroidRuntime: 	at androidx.activity.d.run(SourceFile:118)
```
分析具体异常在哪里呢，需要用到如下SDK中 proguardgui 工具  
`java -jar "D:\android\sdk\tools\proguard\lib\proguardgui.jar"`  
运行后如下图所示
![](https://pic.chenjim.com/202402252353801.png-blog)  
1. 选择 `app\build\outputs\mapping\release\mapping.txt` , 也就是混淆对应文件，每个 release apk 的 mapping.txt 会有所不同，注意备份，参见上图
2. 移除日志中时间、进程ID、TAG等，**只保留每行的后部分**，填入输入框，见上图标注 2。
3. 点击下面的 `reTrace`, 即可看到上图中标注 3 位置显示详细 Trace 信息。


---

逆向反编译可使用工具如下，详细使用按键其官方说明    
<https://github.com/skylot/jadx>  
<https://ibotpeaches.github.io/Apktool/>  
<https://github.com/pxb1988/dex2jar>  
<http://java-decompiler.github.io/>  

---

## 结尾 

本文到这里就结束了，主要介绍了安卓代码混淆的配置和使用，以及如何从混淆后的 Crash 日志中定位源码中位置。在实际使用中，即使开启代码、资源混淆，依然还是可能会被反编译逆向，进而泄露我们软件的实现，可以通过加壳，进一步加固发布的APK，后续会单独进一步介绍。

---

参考文章  
<https://blog.csdn.net/weixin_43632667/article/details/104394222>  
<https://developer.android.com/studio/build/shrink-code>  

--- 

相关文章   
[Android 性能优化--APK加固（1）混淆](https://h89.cn/archives/211.html)  
[Android 性能优化--APK加固（2）加壳](https://h89.cn/archives/212.html)  


