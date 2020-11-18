


# Android 性能优化--APP启动、UI优化

>本文地址 <https://blog.csdn.net/CSqingchen/article/details/106186210>

## 安卓系统启动流程
1. 打开电源，引导芯片代码加载引导程序Boot Loader到RAM中去执行
2. BootLoader把操作系统拉起来
3. Linux 内核启动开始系统设置，找到一个init.rc文件启动初始化进程
4. init进程初始化和启动属性服务，之后开启Zygote进程
5. Zygote开始创建JVM并注册JNI方法，开启SystemServer
6. 启动Binder线程沲和SystemServiceManager,并启动各种服务
7. AMS启动Launcher

## Activity启动流程
大致流程如下  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200616143340794.png)  
详细可以参考 [Activity启动流程源码分析](https://blog.csdn.net/qq_23547831/article/details/51224992)

## 优化启动的Activity
### Activity的Theme优化
在老的系统版本采用以下主题，会引起黑白屏问题，现在的版本默认使用透明处理
```
文件   res/values/styles.xml
白屏    <style name="AppTheme" parent="Theme.AppCompat.Light">
黑屏    <style name="AppTheme">
```
解决办法：自定义启动Activity的主题，比如
```
<style name="AppTheme.Launcher">
     <item name="android:windowDisablePreview">true</item>
    <item name="android:windowBackground">@null</item>
</style>
```
### Activity的布局优化
#### Button(View)的显示过程
1. LayoutInflater加载布局文件到内存，转化为包含大小位置等信息的Button对象
2. CPU经过计算，处理成多为矢量图
3. GPU格栅化[(参考'图像渲染机制')](https://www.jianshu.com/p/1998182670fb) 进行像素填充显示

#### 布局优化方案
- 减少过度绘制，我应该减少红色的显示，比如：  
  减少背景重复，如“Activity的Theme优化”；  
  使用裁减减少控件之间的重合部分；  
  Android7.0之后invalidate()不再执行测量和布局动作。   
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061800312382.png)

- 减少布局层级嵌套，采用`ConstraintLayout`、`<merge>`
- 采用`<include>`复用布局，可以减少GPU重复工作
- 采用`ViewStub`按需加载

#### 常用优化工具
1. GPU过度绘制查看
   手机开发者模式-->调试GPU过度绘制-->显示过度绘制区域。
2. 查看UI层级 `sdk\tools\bin\uiautomatorviewer.bat`
   <img src="https://pic.chenjim.com/20200617235612.png-blog" width = "800" />

3. 查看UI层级嵌套及性能 `sdk\tools\monitor.bat`  
   点击下图三色圆叠加地方后，可以分别显示页面measure、layout、draw的时间  
   红色性能最差，绿色最快。
   <img src="https://pic.chenjim.com/20200618001609.png-blog" width = "800"/>

4. 这里的工具是基于eclipse的，最新的SDK默认已经没有`sdk\tools\`目录，但可参考下图更新SDK工具包  
   <img src="https://pic.chenjim.com/20201203093648543.png-blog" width = "800"/>
   Android Studio 本身也带了"Layout inspector"可查看，如下图：  
   <img src="https://img-blog.csdnimg.cn/20210311172237478.png" width = "800"/>


### Activity的代码优化
- 开启混淆
- 生命周期内方法避免耗时操作
## 优化Application初始化
>一般我们会在onCreate()中放一些初始化，如果初始化耗时，就会引起应用启动慢

优化方案：
- 不放耗时初始化
- 异步初始化非必须业务
- 将部分初始放启动页(不仅仅是为了显示广告)

## Java代码性能分析方法
### logcat查看启动时间
` adb logcat -v time |grep com.xxx.xxx`
查看logcat中包名"com.xxx.xxx"的相关日志

### Profiler查看启动时间
- 在要调试的代码前后添加如下代码，启动应用，会在`data/data/com.xxx.xx/files/app.trace`中生成调试的trace信息
```
val file = File(filesDir, "app.trace")
Debug.startMethodTracing(file.absolutePath)
.....//要监控的代码块
Debug.stopMethodTracing()
```

- 将以上文件导出，并用AS打开，可以进一步分析耗时信息  
  `adb pull data/data/com.xxx.xxxx/files/app.trace`  
  <img src="https://pic.chenjim.com/20200618002603.png-blog" width = "800"/>

**原创文章，转载请注明出处、原文链接！**  
**邮件 <me@h89.cn> ，主页 [https://chenjim.com](https://h89.cn)**
