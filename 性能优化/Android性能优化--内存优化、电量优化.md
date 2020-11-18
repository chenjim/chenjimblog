
@[toc]
#### 内存泄漏的检测、几种常见场景及解决方法
场景一：静态变量持有了Activity  
场景二：匿名内部类或非静态内部类引起的内存泄漏  
场景三：Handler.sendMessageDelayed  
场景四：Static Views  
场景五：无限循环的属性动画  
luyanjun07@内存泄漏的检测、几种常见场景及解决方法
<http://blog.csdn.net/luyanjun07/article/details/58599055>


`adb shelldumpsys meminfo （pid name） ` 查看activities数目  
排查内存泄漏最简单和直观的方法  
<http://www.51testing.com/html/66/n-3714266.html>  
`Android Studio-->profiler memory -->dump java head--> *.hprof`


-----
#### 如何保持设备唤醒状态
原文 <https://www.jianshu.com/p/b49296320f4d>  
耗电分析工具 <https://github.com/google/battery-historian>  

1. 保持屏幕常亮：  
- 方法一：Activity添加getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);    
- 方法二：布局加`android:keepScreenOn="true"`  
2. 使用`PowerManager.WakeLock`保持CPU运行  
 ~~`WakefulBroadcastReceiver`~~ Android O开始已废弃，推荐android.app.job.JobScheduler  
 3. AlarmManager 唤醒CPU，RTC (Real Time Clock) 是一个独立的硬件时钟，可以在 CPU 休眠时正常运行，在预设的时间到达时，通过中断唤醒 CPU。  

#### 电量优化的一些建议
原文 <https://www.jianshu.com/p/b49296320f4d>
1. 充电时执行任务  
2. 连接Wifi后执行任务  
3. 即使释放WakeLock  
4. 大量高频次的CPU唤醒及操作，使用JobScheduler集中处理  
5. 定位完成，及时关闭，如需实时定位可减少更新频率  
6. 网络优化，可参考 <https://blog.csdn.net/CSqingchen/article/details/106671184>