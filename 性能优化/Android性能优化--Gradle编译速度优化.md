
[TOC]

## 1. 保持工具最新  
Android 工具几乎每次更新都会获得构建优化和新功能，保持最新版本可以加快构建速度    
[Android Gradle 插件](https://developer.android.com/build/releases/gradle-plugin)  
[Android Studio 和 SDK 工具](https://developer.android.com/studio/intro/update)  

## 2. 使用 KSP 代替 kapt  
[Kapt（Kotlin 注释处理工具）](https://kotlinlang.org/docs/kapt.html)允许您将 Java 注释处理器与 Kotlin 代码结合使用，即使这些处理器没有对 Kotlin 的特定支持。这是通过从 Kotlin 文件生成 Java 存根来完成的，然后处理器可以读取这些存根。这种存根生成是一项昂贵的操作，并且对构建速度有重大影响。  
[KSP（Kotlin 符号处理）](https://github.com/google/ksp)是 kapt 的 Kotlin 优先替代品。KSP 直接分析 Kotlin 代码，速度提高了 2 倍。它还可以更好地理解 Kotlin 的语言结构。  
建议尽可能从 kapt 迁移到 KSP。在大多数情况下，此迁移只需要更改项目的构建配置。  
如果你的项目使用了[数据绑定(dataBinding)](https://developer.android.com/topic/libraries/data-binding) ，[暂无无法迁移到KSP，尚未计划对 KSP 支持](https://issuetracker.google.com/issues/173030256#comment10) , 可以通过将dataBinding的使用隔离到单独的模块来减轻 kapt 对构建的影响。   
许多流行的库（比如 [Glide](https://bumptech.github.io/glide/doc/download-setup.html#kotlin---ksp)、[Room](https://developer.android.com/jetpack/androidx/releases/room#declaring_dependencies) 等）都已经具有 KSP 支持，还有一些正在添加支持，比如 [Dagger](https://github.com/google/dagger/issues/2349)。   
可参考 [迁移KSP文档](https://developer.android.com/build/migrate-to-ksp) 

## 3. 避免编译不必要的资源  
避免编译和打包未测试的资源，例如其他语言本地化和屏幕密度资源。  
```groovy
android {
    productFlavors {
        create("dev") {
            // 如下配置显示 dev 渠道包，只打包 en 语言和 xxhdpi 屏幕密度的资源  
            resourceConfigurations("en", "xxhdpi")
}   }   }
```
或者如下修改  
```groovy
android {
    applicationVariants.configureEach { variant ->
        if (variant.buildType.name == "debug") {
            variant.mergedFlavor.resourceConfigurations.clear()
            variant.mergedFlavor.resourceConfigurations.add("zh-rCN")
            variant.mergedFlavor.resourceConfigurations.add("xxhdpi")
}     }    }
```
## 4. 优化 repositories maven 排序
如下图，可以查看到需要的依赖库下载结果  
![](https://pic.chenjim.com/202308041547192.png-blog)  
可以将多数能找到放最前面，比如在国内可以使用如下示例  
```
repositories {
    maven { url 'https://maven.aliyun.com/repository/google' }
    maven { url 'https://maven.aliyun.com/repository/public' }
    maven { url 'https://your.private.io/repository' }
    mavenCentral()
    google()
}
```

## 5. 在调试构建中使用静态构建配置值  
使用动态版本代码、版本名称、资源或任何其他更改清单文件的构建逻辑，  
每次您想要运行更改时，都需要完整的应用程序构建，即使实际的更改可能只需要热交换。  
如果 Release 版本确实需要，我们可以只跟改配置，使 debug 版本不生效。  
比如在 BuildConfig 中版本号使用动态时间戳(精确到时分秒)，导致小的修改，大编译，严重影响Rebuild速度。  
[在 Gradle 8.0 默认不生成 BuildConfig.java](https://developer.android.com/build/releases/past-releases/agp-8-0-0-release-notes#default-changes)，进而无法直接引用 BuildConfig 中配置。  

## 6. 使用静态依赖版本  
在build.gradle文件中声明依赖项时，避免使用动态版本号  
(末尾带有加号的版本号，如'androidx.appcompat:appcompat:1.+')  
可能会导致意外的版本更新、难以解决版本差异以及 Gradle 检查更新导致的构建速度变慢。   


## 7. 创建库模块
查找可以转换为 Android 库模块的代码。  
以这种方式模块化您的代码，允许构建系统仅编译您修改的模块，并缓存这些输出以供将来构建。  
模块化还可以使 [并行项目执行更加有效](https://docs.gradle.org/current/userguide/performance.html#parallel_execution)  

## 8. 为自定义构建逻辑创建任务  
编译项目(Make Project)完成后，如下图，点击 `Build Analyzer`   
![](https://pic.chenjim.com/202308041550159.png-blog)  
会跳转到编译分析栏，会显示，Config 及 Task 执行时间，如下图  
![](https://pic.chenjim.com/202308041553188.png-blog)  
切换 `Overview` 为 `Tasks`，可以显示各个 Task 执行耗时 
![](https://pic.chenjim.com/202308041559775.png-blog)  
创建构建配置文件后，如果构建配置文件显示相对较长的构建时间花费在 `Configuration` 阶段，  
请检查您的build.gradle脚本，并查找要包含在自定义 Gradle 任务中的代码，  
[通过将一些构建逻辑移至任务中，您可以帮助确保任务仅在需要时运行，可以缓存结果以供后续构建使用](https://docs.gradle.org/current/userguide/performance.html#enable_incremental_build_for_custom_tasks)

## 9. 将图像转换为 WebP 
WebP 可以提供比 JPEG 或 PNG 更好的压缩。
减少图像文件大小而不必执行构建时压缩可以加快构建速度，特别是当您的应用程序使用大量图像资源时。  
使用 Android Studio 可以 [轻松 将图像转换为 WebP](https://developer.android.com/studio/write/convert-webp#convert_images_to_webp)。  

## 10. 禁用 PNG 处理
如果您不将 PNG 图像转换为 WebP，您仍然可以通过在每次构建应用程序时禁用自动图像压缩来加快构建速度。  
如果您使用的是 [Android Gradle 插件 3.0.0](https://developer.android.com/studio/releases/gradle-plugin#3-0-0) 或更高版本，则默认情况下，“debug”构建类型会禁用 PNG 处理。
如果 Release 也需要禁用 PNG 处理，可以参见如下修改  
```groovy
android {
    buildTypes {
        release {
            crunchPngs false
            //  isCrunchPngs = false //for kotlin
}   }   }
```

## 11. 挑选 JVM 垃圾收集器
通过配置 Gradle 使用的最佳 JVM 垃圾收集器可以提高构建性能。  
虽然 JDK 8 默认配置为使用并行垃圾收集器，但 JDK 9 及更高版本配置为使用 [G1 垃圾收集器](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-ED3AB6D3-FD9B-4447-9EDF-983ED2F7A573)。  
建议 使用并行垃圾收集器测试您的 Gradle 构建，需要在 `gradle.properties` 添加如下修改  
`org.gradle.jvmargs=-XX:+UseParallelGC` 或者 `org.gradle.jvmargs=-Xmx4608M -XX:+UseParallelGC`


## 12. 增加 JVM 堆大小
在 编译分析 栏，我们可以看到编译过程中，GC 花费的构建时间，  
如果 超过 超过 15% ，我们可以增加 Java 虚拟机 (JVM) 堆大小  
需要在 `gradle.properties` 中修改限制 4、6 或 8 GB ，参考如下  
`org.gradle.jvmargs=-Xmx6g`  

## 13. 使用非传递 R 类

使用非传递R类可以更快地构建具有多个模块的应用程序。  
这样做可以确保每个模块的R类仅包含对其自身资源的引用，而不从其依赖项中提取引用，从而有助于防止资源重复。  
这会带来更快的构建速度以及避免编译的相应好处。  
这是 Android Gradle 插件 8.0.0 及更高版本中的默认行为。

## 14. 使用非常量 R 类
在APP模块和自动化测试时，使用非常量的 R.class 可以提高 Java 编译的增量并允许更精确的资源收缩。  
在Library模块，R.class 一直不是常量，只有在打包APP或者自动化测试时，资源才被编号。  
这是 Android Gradle Plugin 8.0.0 及更高版本中的默认行为，也就是如下代码会编译出错.  
```java
switch (viewId) {
    case R.id.bt_back:
        dosth1;
        break;
    case R.id.bt_enter:
        dosth2();
        break;
}

// 可以替换为  
if (viewId == R.id.bt_back) dosth1;
else if (viewId == R.id.bt_enter) dosth2();
```

## 15. 禁用 Jetifier 标志
由于大多数项目直接使用 AndroidX 库，因此您可以删除 Jetifier标志以获得更好的构建性能。  
在 `gradle.properties` 修改如下即可  
`android.enableJetifier=false`  

## 16. 使用配置缓存
[配置缓存](https://docs.gradle.org/current/userguide/configuration_cache.html) 允许 Gradle 记录有关构建任务图的信息并在后续构建中重用它，不必再次重新配置整个构建。  
可以在 `gradle.properties` 修改如下  
```
org.gradle.caching=true
org.gradle.configuration-cache=true
org.gradle.configuration-cache.problems=warn
```
注意可能会存在冲突的情况，参见 [Gradle#13490](https://github.com/gradle/gradle/issues/13490)  
启用配置缓存后，第一次运行项目时，构建输出会显示  
`Calculating task graph as no configuration cache is available for tasks`   
在后续运行期间，构建输出显示  
`Reusing configuration cache`   

---------

参考文档  
<https://developer.android.com/build/optimize-your-build>  
<https://developer.android.com/build/migrate-to-ksp>  



