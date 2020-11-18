
@[toc]


# 知识导航-Android第三方框架使用及原理解析

>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/106355205>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>  

### 图片加载

#### Universal-Image-Loader
[Universal-Image-Loader完全解析（一）--- 基本介绍及使用](http://blog.csdn.net/xiaanming/article/details/26810303)
[Universal-Image-Loader完全解析（二）--- 图片缓存策略详解](http://blog.csdn.net/xiaanming/article/details/27525741)
[Universal-Image-Loader完全解析（三）---源代码解读](http://blog.csdn.net/xiaanming/article/details/39057201)


---

#### Glide
- 开源地址 <https://github.com/bumptech/glide> 
- 使用文档 <https://muyangmin.github.io/glide-docs-cn/>
- Glide 加载原理  <https://www.jianshu.com/p/9d8aeaa5a329>
- 缓存机制 <https://www.jianshu.com/p/17644406396b>
- Glide系列  嘎啦果安卓兽@简书 <https://www.jianshu.com/nb/23459686>
- Glide LruCache使用及源码解析 <https://juejin.im/post/5e535a4b518825496452b063>

---

#### 图片加载框架对比
<https://www.jianshu.com/p/89ab4f415bf8>

### 图片伸缩
<https://github.com/davemorrissey/subsampling-scale-image-view>

---

### View 相关

#### RecyclerView
- 自定义LayoutManager，RecyclerView中如何自定义LayoutManager


####  VLayout实现原理，即如何自定义LayoutManager

#### DialogV3 安全简洁易用
相关介绍：<https://github.com/kongzue/DialogV3>


----

### 网络相关

#### OKHttp3的使用，网络请求中的Intercept

Android之网络请求1————HTTP协议  
<https://blog.csdn.net/qq_38499859/article/details/82153094>   

Android之网络请求2————OkHttp的基本使用  
<https://blog.csdn.net/qq_38499859/article/details/82290738>  


Android之网络请求3————OkHttp的拦截器和封装  
<https://blog.csdn.net/qq_38499859/article/details/82355954>  

Android之网络请求4————OkHttp源码1:框架  
<https://blog.csdn.net/qq_38499859/article/details/82469295>   

Android之网络请求5————OkHttp源码2:发送请求   
<https://blog.csdn.net/qq_38499859/article/details/82561675>  

Android之网络请求6————OkHttp源码3:拦截器链  
<https://blog.csdn.net/qq_38499859/article/details/82630630>   


Android之网络请求7————OkHttp源码4:网络操作   
<https://blog.csdn.net/qq_38499859/article/details/82745671>   

Android之网络请求8————OkHttp源码5:缓存相关   
<https://blog.csdn.net/qq_38499859/article/details/82778955>    

#### Retrofit  的实现与原理
Retrofit 在 OkHttp 上做了哪些封装？动态代理和静态代理的区别，是怎么实现的

Android之网络请求9————Retrofit的简单使用  
<https://blog.csdn.net/qq_38499859/article/details/82807496>  


Android之网络请求10————Retrofit的进阶使用  
<https://blog.csdn.net/qq_38499859/article/details/83010604>  


Android之网络请求11————Retrofit的源码分析  
<https://blog.csdn.net/qq_38499859/article/details/83042782>  
上文有下图(盗图一张，不要打我)   
![](https://pic.chenjim.com/20211009114312.png-blog)  

---

### 架构设计相关

---

#### EventBus实现原理  
<https://blog.csdn.net/u013309870/article/details/105438715>  

**总结**  
1. 要理解EventBus就要从register，unRegister,post,postSticky方法入手。要理解register实质上是将订阅对象（比如activity）中的每个带有subscriber的方法找出来，最后获得调用的就是这些方法。订阅对象（比如activity）是一组event方法的持有者。

2. 后注册的对象中sticky方法能够收到之前的stickyEvent方法的原因是EventBus中维护了stickyEvent的hashMap表，在subsribe注册的时候就遍历其中有没有注册监听stickyEvent如果有就会执行一次回调。

**EventBus缺点**  
1. 使用的时候有定义很多event类，
2. event在注册的时候会调用反射去遍历注册对象的方法在其中找出带有@subscriber标签的方法，性能不高。
3. 需要自己注册和反注册，如果忘了反注册就会导致内存泄漏


---

#### 用LiveDataBus替代RxBus、EventBus
Androi消息总线的演进之路：用[LiveDataBus](https://github.com/JeremyLiao/LiveEventBus)替代RxBus、EventBus  
<https://tech.meituan.com/2018/07/26/android-livedatabus.html>  
| 消息总线     | 延迟发送 | 有序接收消息 | Sticky | 生命周期感知 | 跨进程/APP | 线程分发 |
| ------------ | -------- | ------------ | ------ | ------------ | ---------- | -------- |
| EventBus     | ❌        | ✅            | ✅      | ❌            | ❌          | ✅        |
| RxBus        |  ❌        | ❌            | ✅      | ❌            | ❌          | ✅        |
| LiveEventBus | ✅        | ✅            | ✅      | ✅            | ✅          | ❌        |

---
#### ButterKnife实现原理  
<https://juejin.cn/post/6844903495405928456> 
1. ButterKnife首先会在编译期间利用Java Annotation Processin生成一个XXX_ViewBinding的类。
2. ButterKnife.bind中通过反射实例化XX_ViewBinding，完成View的各项绑定，实现方式就是普通的findviewbyid等。
3. unbinder.unbind方法是一些置空操作，便于垃圾回收器回收内存。


----

#### RxJava实现原理
  <https://blog.csdn.net/sinat_23092639/article/details/104356256>  
  

RxJava 的线程切换原理

#### Dagger2
Dagger2@github: <https://github.com/google/dagger>
Dagger2@demo:<https://github.com/google/dagger/tree/master/examples/simple>

1. [Dagger2从入门到放弃再到恍然大悟(MVP设计)](http://www.jianshu.com/p/39d1df6c877d)
2. [详解Dagger2]( http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0519/2892.html)
3. [Dagger2 这次入门就不用放弃了](http://blog.csdn.net/u012943767/article/details/51897247)



---
#### LeakCanary

#### 组件化开发 
- ARouter 
ARouter使用 @Github   <https://github.com/alibaba/ARouter>
源码解析 <https://www.jianshu.com/p/5b35309e9bb2>

- CC实现原理
CC@github <https://github.com/luckybilly/CC>  
关于CC <https://qibilly.com/CC-website/#/about>  
CC 使用 <https://qibilly.com/CC-website/#/integration>  
CC原理 <https://qibilly.com/CC-website/#/article-cc-principle>  

#### 插件化开发 
- Android插件化原理解析
<https://www.jianshu.com/p/d3231a15afee>
示例 <https://github.com/xch168/PluginDemo>
- 常见android插件框架对比
  <https://blog.csdn.net/CSqingchen/article/details/106540681>
- RePlugin流程与源码解析
  <https://www.jianshu.com/p/18530be5dcdd>

---

#### 热修复实现原理，解决方案
- 美团 Robust  <https://github.com/Meituan-Dianping/Robust/blob/master/README-zh.md>
- 腾讯  Tinker <https://github.com/Tencent/tinker>
- 阿里 <https://github.com/alibaba/AndFix> (很久未维护)

[安卓App热补丁动态修复技术介绍 by QQ空间team](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a)

参考自 <http://blog.csdn.net/lmj623565791/article/details/49883661>

#### 多渠道打包方案及原理

- 腾讯的VasDolly <https://github.com/Tencent/VasDolly>
  VasDolly 原理 <https://github.com/Tencent/VasDolly/wiki/VasDolly%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86>

- 美团的 walle <https://github.com/Meituan-Dianping/walle>
  walle 原理 <https://tech.meituan.com/2017/01/13/android-apk-v2-signature-scheme.html>

|  多渠道打包工具对比   | VasDolly | packer-ng-plugin | Walle  |
| :-------------------: | :------: | :--------------: | :----: |
|      V1签名方案       |   支持   |       支持       | 不支持 |
|      V2签名方案       |   支持   |      不支持      |  支持  |
|    已有注释块的APK    |   支持   |      不支持      | 不支持 |
| 根据已有APK生成渠道包 |   支持   |      不支持      | 不支持 |
|      命令行工具       |   支持   |       支持       |  支持  |
|        强校验         |   支持   |      不支持      | 不支持 |
|    多线程加速打包     |   支持   |      不支持      | 不支持 |
####   Android WebView独立进程解决方案

<https://www.jianshu.com/p/b66c225c19e2>
Github:源码 <https://github.com/xudjx/webprogress>


#### Android进程通信以及多进程

- [多进程通信解决方案：爱奇艺 Andromeda](https://mp.weixin.qq.com/s/PZs1wss3PizqSE8U2RGXYw)


#### Fragment 管理框架  
  <https://github.com/weikaiyun/SFragmentation>  


---

**原创文章，转载请注明出处、原文链接！<me@h89.cn>   我的主页<https://chenjim.com>**

---

相关系列文章推荐
- [算法与数据结构](https://blog.csdn.net/CSqingchen/article/details/106357307)
- [Java基础知识点总结](https://blog.csdn.net/CSqingchen/article/details/106356406)
- [Android 基础知识点相关](https://blog.csdn.net/CSqingchen/article/details/106391013)
- [Android View 相关](https://blog.csdn.net/CSqingchen/article/details/106380388)
- [Android APP 架构设计相关](https://blog.csdn.net/CSqingchen/article/details/106355327)
- [Android 第三方框架的原理解析相关](https://blog.csdn.net/CSqingchen/article/details/106355205)
- [好用的Android开源框架整理](https://github.com/Trinea/android-open-project) 
- [Android 网络 相关](https://blog.csdn.net/CSqingchen/article/details/106362599)
- [Android Flutter 相关](https://blog.csdn.net/CSqingchen/article/details/106362578)
- [Android Kotlin 相关](https://blog.csdn.net/CSqingchen/article/details/106362566)
- [Android 性能优化](https://blog.csdn.net/csqingchen/category_10025139.html)

---