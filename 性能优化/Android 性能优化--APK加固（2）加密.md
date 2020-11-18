
[TOC]

> 本文首发地址：<https://h89.cn/archives/212.html>  
> 最新更新地址：<https://gitee.com/chenjim/chenjimblog>

通过 [前文](https://h89.cn/archives/211.html) 介绍，我们知晓了如何使用代码混淆和资源混淆加固我们的APK，以及如何分析混淆后Crash日志问题。本文将进一步介绍APP加固的相关方法，比如字符串加密、资源加密、签名效验、DEX加密等。

## 字符串加密

反编译 [ProguardDemo.apk](https://gitee.com/chenjim/ProguardDemo/blob/main/out/app-release.apk) 可以看到字符串`chenjim`，如下图，可能会泄露我们的重要信息    
![](https://pic.chenjim.com/202402262216496.png-blog)  

[StringFog](https://github.com/MegatronKing/StringFog) 提供了一个很好的方案：  
编译时对所有字符串进行加密，运行时进行解密，可以自定义加解密算法。  

## 图片加密

通过反编译后，我们能看到所有使用图片的资源文件，如何避免被盗用呢？  
可以对图片进行解密，然后放到Assets目录，使用时先解密，再加载显示。  
既然后都显示出来了，还是可以被截图等方式盗用。  
因此图片加密使用的不多，有点画蛇添足，没有具体源码实现。  

## 如何避免应用被重新签名分发
如果应用被逆向加入其他程序，很容易造成其他严重后果，我们可以在应用中加入签名的效验，如果不满足提示或者直接退出应用。  
[CSDN 博文](https://blog.csdn.net/weixin_43632667/article/details/104394222) 中给出了 JAVA 和 JNI 获取应用签名SHA1的方法。


## APK 加壳的方案简析

- 方案一：直接对apk进行加密，启动应用时通过壳程序去加载apk运行，会有性能损失；
  他们的原理是给我们的应用加一层壳，直接反编译得到是加固的壳，从而保护我们发布的APK。  
  Github上有许多实现方案，如下    
      <https://github.com/guanchao/apk_auto_enforce>   
      <https://github.com/yongyecc/dexshellerInMemory>     
      <https://github.com/Frezrik/Jiagu>   
  目前三方方案也都比较成熟，如 [腾讯T-Sec](https://cloud.tencent.com/product/ms)、[360加固](https://jiagu.360.cn/)、[网易易盾](https://dun.163.com/product/android-reinforce) 等
- 方式二：仅对原apk的dex文件进行加密，启动应用时对dex解密，通过DexClassLoader进行加载，目前多数也是这么实现的；。


---

## DEX加密原理及实现
原计划是介绍DEX加密详细原理及实现，看到 [韩曙亮](https://hanshuliang.blog.csdn.net/) 有相关博文进行介绍，本节先列举相关文章链接，后续单独写一篇内容总结。  
- [Android App加固原理与技术历程 By Security-X@CSDN ](https://blog.csdn.net/estelll/article/details/106236765)  
  ![](https://pic.chenjim.com/202402282200546.png-blog)
- [DEX 加密 ( 常用 Android 反编译工具 | apktool | dex2jar | enjarify | jd-gui | jadx )](https://hanshuliang.blog.csdn.net/article/details/109540997)
- [DEX 加密 ( DEX 加密原理 | DEX 加密简介 | APK 文件分析 | DEX 分割 )](https://hanshuliang.blog.csdn.net/article/details/109596454)
- [DEX 加密 ( 多 DEX 加载 | 65535 方法数限制和 MultiDex 配置 | PathClassLoader 类加载源码分析 | DexPathList )](https://hanshuliang.blog.csdn.net/article/details/109605996)
- [DEX 加密 ( 不同 Android 版本的 DEX 加载 | Android 8.0 版本 DEX 加载分析 | Android 5.0 版本 DEX 加载分析 )](https://hanshuliang.blog.csdn.net/article/details/109608605)
- [DEX 加密 ( DEX 加密使用到的相关工具 | dx 工具 | zipalign 对齐工具 | apksigner 签名工具 )](https://hanshuliang.blog.csdn.net/article/details/109612510)
- [DEX 加密 ( 支持多 DEX 的 Android 工程结构 )](https://hanshuliang.blog.csdn.net/article/details/109629838)
- [DEX 加密 ( 代理 Application 开发 | multiple-dex-core 依赖库开发 | 配置元数据 | 获取 apk 文件并准备相关目录 )](https://hanshuliang.blog.csdn.net/article/details/109649242)
- [DEX 加密 ( 代理 Application 开发 | 解压 apk 文件 | 判定是否是第一次启动 | 递归删除文件操作 | 解压 Zip 文件操作 )](https://hanshuliang.blog.csdn.net/article/details/109700882)
- [DEX 加密 ( 代理 Application 开发 | 加载 dex 文件 | 反射获取系统的 Element[] dexElements )](https://hanshuliang.blog.csdn.net/article/details/109706787)
- [DEX 加密 ( 代理 Application 开发 | 加载 dex 文件 | 使用反射获取方法创建本应用的 dexElements | 各版本创建 dex 数组源码对比 )](https://hanshuliang.blog.csdn.net/article/details/109720729)
- [DEX 加密 ( 代理 Application 开发 | 加载 dex 文件 | 将系统的 dexElements 与 应用的 dexElements 合并 | 替换操作 )](https://hanshuliang.blog.csdn.net/article/details/109769126)
- [DEX 加密 ( 代理 Application 开发 | 交叉编译 OpenSSL 开源库 )](https://hanshuliang.blog.csdn.net/article/details/109844305)
- [DEX 加密 ( 代理 Application 开发 | 项目中配置 OpenSSL 开源库 | 使用 OpenSSL 开源库解密 dex 文件 )](https://hanshuliang.blog.csdn.net/article/details/109958355)
- [DEX 加密 ( 阶段总结 | 主应用 | 代理 Application | Java 工具 | 代码示例 ) ★](https://hanshuliang.blog.csdn.net/article/details/110450891)
- [DEX 加密 ( Java 工具开发 | 加密解密算法 API | 编译代理 Application 依赖库 | 解压依赖库 aar 文件 )](https://hanshuliang.blog.csdn.net/article/details/109783011)
- [DEX 加密 ( Java 工具开发 | 生成 dex 文件 | Java 命令行执行 )](https://hanshuliang.blog.csdn.net/article/details/109801199)
- [DEX 加密 ( Java 工具开发 | 解压 apk 文件 | 加密生成 dex 文件 | 打包未签名 apk 文件 | 文件解压缩相关代码 )](https://hanshuliang.blog.csdn.net/article/details/109810588)
- [DEX 加密 ( Java 工具开发 | apk 文件对齐 )](https://hanshuliang.blog.csdn.net/article/details/109823308)
- [DEX 加密 ( Java 工具开发 | apk 文件签名 )](https://hanshuliang.blog.csdn.net/article/details/109842159)
- [DEX 加密 ( 阶段总结 | 主应用 | 代理 Application | Java 工具 | 代码示例 ) ★](https://hanshuliang.blog.csdn.net/article/details/110450891)
- [DEX 加密 ( Application 替换 | Android 应用启动原理 )](https://hanshuliang.blog.csdn.net/article/details/109995592)
- [DEX 加密 ( Application 替换 | Android 应用启动原理 | ActivityThread 源码分析 )](https://hanshuliang.blog.csdn.net/article/details/110090925)
- [DEX 加密 ( Application 替换 | Android 应用启动原理 | LoadedApk 源码分析 )](https://hanshuliang.blog.csdn.net/article/details/110093625)
- [DEX 加密 ( Application 替换 | Android 应用启动原理 | Instrumentation 源码分析 )](https://hanshuliang.blog.csdn.net/article/details/110245437)
- [DEX 加密 ( Application 替换 | Android 应用启动原理 | LoadedApk 后续分析 )](https://hanshuliang.blog.csdn.net/article/details/110261362)
- [DEX 加密 ( Application 替换 | Android 应用启动原理 | ActivityThread 后续分析 | Application 替换位置 )](https://hanshuliang.blog.csdn.net/article/details/110261493)
- [DEX 加密 ( Application 替换 | 获取 ContextImpl、ActivityThread、LoadedApk 类型对象 | 源码分析 )](https://hanshuliang.blog.csdn.net/article/details/110264099)
- [DEX 加密 ( Application 替换 | 获取 ContextImpl、ActivityThread、LoadedApk 类型对象 )](https://hanshuliang.blog.csdn.net/article/details/110304568)
- [DEX 加密 ( Application 替换 | 判定自定义 Application 存在 | 获取 ContextImpl 对象 )](https://hanshuliang.blog.csdn.net/article/details/111569017)
- [DEX 加密 ( Application 替换 | 创建用户自定义 Application | 替换 ContextImpl 对象的 mOuterContext 成员 )](https://hanshuliang.blog.csdn.net/article/details/111569352)
- [DEX 加密 ( Application 替换 | 加密不侵入原则 | 替换 ActivityThread 的 mInitialApplication 成员 )](https://hanshuliang.blog.csdn.net/article/details/111567202)
- [DEX 加密 ( Application 替换 | ActivityThread 中的 mAllApplications 集合添加 Application )](https://hanshuliang.blog.csdn.net/article/details/115392443)
- [DEX 加密 ( Application 替换 | 替换 LoadedApk 中的 Application mApplication 成员 )](https://hanshuliang.blog.csdn.net/article/details/115393345)
- [DEX 加密 ( Application 替换 | 修改 LoadedApk 中的 mApplicationInfo 成员的 className 名称 )](https://hanshuliang.blog.csdn.net/article/details/115407209)
- [DEX 加密 ( Application 替换 | 分析 Activity 组件中获取的 Application | ActivityThread | LoadedApk )](https://hanshuliang.blog.csdn.net/article/details/115407466)
- [DEX 加密 ( Application 替换 | 分析 Service 组件中调用 getApplication() 获取的 Application 是否替换成功 )](https://hanshuliang.blog.csdn.net/article/details/115408089)
- [DEX 加密 ( Application 替换 | 分析 BroadcastReceiver 组件中调用 getApplication() 获取的 Application )](https://hanshuliang.blog.csdn.net/article/details/115415330)
- [DEX 加密 ( Application 替换 | 分析 ContentProvider 组件中调用 getApplication() 获取的 Application )](https://hanshuliang.blog.csdn.net/article/details/115421115)
- [DEX 加密 ( Application 替换 | 分析 ContentProvider 组件中调用 getApplication() 获取的 Application 二 )](https://hanshuliang.blog.csdn.net/article/details/115436316)
- [DEX 加密 ( Application 替换 | 兼容 ContentProvider 操作 | 源码资源 )](https://hanshuliang.blog.csdn.net/article/details/115444337)
- [Android APK加固原理](https://blog.csdn.net/kingwjh/article/details/128814421)


---

参考文章  
<https://blog.csdn.net/weixin_43632667/article/details/104394222>  
<https://developer.android.com/studio/build/shrink-code>  
[APK 加固总结 By 韩曙亮](https://blog.csdn.net/shulianghan/article/details/115458625)


--- 

相关文章   
[Android 性能优化--APK加固（1）混淆](https://h89.cn/archives/211.html)  
[Android 性能优化--APK加固（2）加密](https://h89.cn/archives/212.html)  