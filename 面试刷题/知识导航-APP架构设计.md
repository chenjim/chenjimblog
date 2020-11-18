

@[toc]

# 知识导航-APP架构设计

>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/106355327>
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>

#### 注解处理器
Android运行时注解的使用 <https://www.jianshu.com/p/de13b00042d6>
Android编译时注解的使用 <https://www.jianshu.com/p/3052fa51ee95>

#### 数据存储--MMKV使用及原理
<https://blog.csdn.net/CSqingchen/article/details/106344399>
<https://github.com/Tencent/MMKV/wiki/android_tutorial_cn>

#### 数据存储--room使用
<https://developer.android.com/training/data-storage/room>
<https://www.jianshu.com/p/3e358eb9ac43>


#### RxAndroid的使用方式
<http://www.jianshu.com/p/031745744bfa>
1 什么是响应式编程  
2 什么是observable  
3 如何将异步事件比如按钮点击或者EditText字符变化转换成observables  
4 observable变换  
5 observable 过滤拦截  
6 如何指定链式中的代码执行线程  
7 如何合并多个observables  

给 Android 开发者的 RxJava 详解: <https://github.com/JackChan1999/RxJava_Docs_For_Android_Developer>

#### 自定义类加载器加载加密类文件
<https://blog.csdn.net/Android_SE/article/details/89923908>

---
#### Android动态化框架App Bundles
 App 动态化框架（即Android App Bundle，缩写为AAB），AAB是借助Split Apk完成动态加载，使用AAB动态下发方式，可以大幅度减少应用体积
<https://www.jianshu.com/p/57cccc680bb6>

---

#### MVC、MVP、MVVM

- [Android MVVM 系列之 Databinding](https://www.jianshu.com/p/4eed670fd689)，参考自[Google Databinding doc](https://developer.android.com/topic/libraries/data-binding/index.html)

- [如何构建Android MVVM应用程序(Databinding)](http://www.jianshu.com/p/2fc41a310f79)

- [Android中MVP设计使用(与MVC区别)](http://blog.csdn.net/lovejjfg/article/details/50813448)

- MVC、MVP、MVVM差别

| **类型** | **创建过程** | **A/F** | **特点**                                        | **缺点**                          | **应用建议**               |
| -------- | ------------ | ------- | ----------------------------------------------- | --------------------------------- | -------------------------- |
| MVC      | C->M + V     | C       | 分离了Model和Controller                         | Controller变得越来越复杂          | 简单的、不大修改的页面     |
| MVP      | V -> P -> M  | V       | 在MVC的基础上通过Interface彻底分离了View和Model | Presenter与View的交互会琐碎而复杂 | 核心、复杂、需求变更快页面 |
| MVVM     | V -> VM -> M | V       | 在MVP的基础上增加了Data Binding, 代码量更小     | XML中包含代码                     | 核心、复杂、需求变更快页面 |


Jetpack-MVVM-Best-Practice(Jetpack MVVM 最佳实践)
<https://github.com/KunMinX/Jetpack-MVVM-Best-Practice>

---

#### 混合开发及Android WebView应用
混合开发涉及到的知识点主要包括：

1.  APP调用WebView加载url
2.  掌握WebView的封装，了解所有的WebSettings配置，掌握WebViewClient、WebChromeClient
3.  掌握WebView和Native双向通信机制，会自己封装双向通信中间件

对WebView的封装可参考： 
[GitHub:AgentWeb](https://github.com/Justson/AgentWeb)

对通信中间件原理理解：  
[GitHub：webprogress](https://github.com/xudjx/webprogress)

---

####  Android屏幕适配全方位解析

- [简书：高级UI---LSN-9-1-android屏幕适配全方位解析](https://www.jianshu.com/p/0586c7e7e212)

- [字节跳动技术团队：一种极低成本的Android屏幕适配方式](https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA)

- [Github: A low-cost Android screen adaptation solution](https://github.com/JessYanCoding/AndroidAutoSize)

----


#### Android中的Apk的加固(加壳)原理解析和实现
<https://blog.csdn.net/zcmain/article/details/72887439>

---
#### Android 中使用AOP
<https://www.jianshu.com/p/83c46664b507>

- 归纳AOP在Android开发中的几种常见用法
  <https://www.jianshu.com/p/2779e3bb1f14>


---

#### Gradle，自动化构建，持续集成相关   [待完善]

---

#### Android Studio编译过程
其中使用到的编译工具：
aapt、aidl、Java Compiler、dex、 zipalign

1.  通过aapt打包res资源文件，生成R.java、resources.arsc和res文件（二进制 & 非二进制如res/raw和pic保持原样）
2.  处理.aidl文件，生成对应的Java接口文件
3.  通过Java Compiler编译R.java、Java接口文件、Java源文件，生成.class文件
4.  通过dex命令，将.class文件和第三方库中的.class文件处理生成classes.dex
5.  通过apkbuilder工具，将aapt生成的resources.arsc和res文件、assets文件和classes.dex一起打包生成apk
6.  通过zipalign工具，将进行对齐处理。
7.  通过Jarsigner工具，对上面的apk进行签名

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNDIwODY2LTc5YjY3NTc2MTAxOGZmNTcucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvNTM2L2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNDIwODY2LTE0MDI0OTRhNTIyNTk2NGUucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvOTkzL2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

---
#### 其它安卓APP架构设计
- [googlesamples/android-architecture· GitHub](https://github.com/googlesamples/android-architecture)
A collection of samples to discuss and showcase different architectural tools and patterns for Android apps
(翻译：一组示例，讨论和展示Android应用程序的不同架构工具和模式)



- [google/iosched · GitHub](https://github.com/google/iosched)，
Google出品，异曲同工，比下面原答案更易学，更优雅。

- [android10/Android-CleanArchitecture · GitHub](https://github.com/android10/Android-CleanArchitecture)

---

####  对移动端架构的思考
*   MVC模式 优缺点
*   MVP模式 优缺点
*   MVVM模式 优缺点
*   CLEAN模式 优缺点
*   组件化开发
*   跨平台开发：Flutter、ReactNative 对比（RN未来要黄，了解一下就好）
原文：<https://mp.weixin.qq.com/s/OEzcsPZHCVkjeUCh6hMuWg>

---

参考自：
1.  [Android 开发有什么好的架构么?@知乎](https://www.zhihu.com/question/21406685/answer/61122814)

**原创文章，转载请注明出处、原文链接！  
邮件<me@h89.cn> 主页[https://chenjim.com](https://h89.cn)**

---

相关系列文章推荐
- [算法与数据结构](https://blog.csdn.net/CSqingchen/article/details/106357307)
- [Java基础知识点总结](https://blog.csdn.net/CSqingchen/article/details/106356406)
- [Android 基础知识点相关](https://blog.csdn.net/CSqingchen/article/details/106391013)
- [Android View 相关](https://blog.csdn.net/CSqingchen/article/details/106380388)
- [Android APP 架构设计相关](https://blog.csdn.net/CSqingchen/article/details/106355327)
- [Android 第三方框架的原理解析相关](https://blog.csdn.net/CSqingchen/article/details/106355205)
- [Android 网络 相关](https://blog.csdn.net/CSqingchen/article/details/106362599)
- [Android Flutter 相关](https://blog.csdn.net/CSqingchen/article/details/106362578)
- [Android Kotlin 相关](https://blog.csdn.net/CSqingchen/article/details/106362566)
- [Android 性能优化](https://blog.csdn.net/csqingchen/category_10025139.html)

---
