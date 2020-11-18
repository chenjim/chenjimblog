**原文** [Android图形系统之VSYNC(十)](https://unbroken.blog.csdn.net/article/details/122410483)  

## 1.VSYNC

VSYNC 信号可同步显示流水线。显示流水线由应用渲染、SurfaceFlinger 合成以及用于在屏幕上显示图像的硬件混合渲染器 (HWC)
组成。VSYNC 可同步应用唤醒以开始渲染的时间、SurfaceFlinger
唤醒以合成屏幕的时间以及屏幕刷新周期。这种同步可以消除卡顿，并提升图形的视觉表现。

HWC 可生成 VSYNC 事件并通过回调将事件发送到 SurfaceFlinger：

  
typedef void (*HWC2_PFN_VSYNC)(hwc2_callback_data_t callbackData,  
hwc2_display_t display, int64_t timestamp);  
注意：HWC 从 HAL_PRIORITY_URGENT_DISPLAY 的线程触发 hwc2_callback_data_t，延迟时间尽可能短，通常小于
0.5 毫秒。  
SurfaceFlinger 通过调用 setVsyncEnabled 来控制 HWC 是否生成 VSYNC 事件。SurfaceFlinger 使
setVsyncEnabled 能够生成 VSYNC 事件，因此它可以与屏幕的刷新周期同步。当 SurfaceFlinger
同步到屏幕刷新周期时，SurfaceFlinger 会停用 setVsyncEnabled 以阻止 HWC 生成 VSYNC 事件。如果
SurfaceFlinger 检测到实际 VSYNC 与它先前建立的 VSYNC 之间存在差异，则 SurfaceFlinger 会重新启动 VSYNC
事件生成过程。

### **VSYNC 偏移**

  
同步应用和 SurfaceFlinger 渲染循环应同步到硬件 VSYNC。在 VSYNC 事件中，屏幕开始显示帧 N，而 SurfaceFlinger
开始为帧 N+1 合成窗口。应用处理等待的输入并生成帧 N+2。

与 VSYNC 同步会实现一致的延迟时间。它可以减少应用和 SurfaceFlinger 中的错误，并最大限度减小相位内外屏幕之间的偏移。这要假定应用和
SurfaceFlinger 的每帧时间没有很大变化。延迟至少为两帧。

为了解决此问题，您可以通过使应用和合成信号与硬件 VSYNC 相关，从而利用 VSYNC
偏移减少输入设备到屏幕的延迟。这是有可能的，因为应用加合成通常需要不到 33 毫秒的时间。

VSYNC 偏移的结果是具有相同周期和偏移相位的三个信号：

HW_VSYNC_0 - 屏幕开始显示下一帧。  
VSYNC - 应用读取输入内容并生成下一帧。  
SF_VSYNC - SurfaceFlinger 开始为下一帧进行合成。  
通过 VSYNC 偏移，SurfaceFlinger 接收缓冲区并合成帧，而应用同时处理输入内容并渲染帧。

注意：VSYNC 偏移会缩短可用于应用和合成的时间，因此增加了出错几率。

###  
**DispSync**

  
DispSync 维护屏幕基于硬件的周期性 VSYNC 事件的模型，并使用该模型在硬件 VSYNC 事件的特定相位偏移处执行回调。

DispSync 是一个软件锁相回路 (PLL)，它可以生成由 Choreographer 和 SurfaceFlinger 使用的 VSYNC 和
SF_VSYNC 信号，即使没有来自硬件 VSYNC 的偏移也是如此。

DispSync 流程

![](https://img-blog.csdnimg.cn/f2c378f959724974be79ea9a987ac38c.png?x-oss-
process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oWi5oWi55qE54eD54On,size_15,color_FFFFFF,t_70,g_se,x_16)  
图 1. DispSync 流程

DispSync 具有以下特点：

参考 - HW_VSYNC_0。  
输出 - VSYNC 和 SF_VSYNC。  
反馈 - 自硬件混合渲染器的退出栅栏有信号状态时间戳。

###  
**VSYNC/退出偏移**

  
退出栅栏的有信号状态时间戳必须与 HW VSYNC
相符，即使在不使用偏移相位的设备上也是如此。否则，造成的错误会更加严重。智能面板通常有一个增量：退出栅栏是对显示内存进行直接内存访问 (DMA)
的终点，但是实际的显示切换和 HW VSYNC 会晚一段时间。

PRESENT_TIME_OFFSET_FROM_VSYNC_NS 在设备的 BoardConfig.mk makefile
中设置。它取决于屏幕控制器和面板特性。从退出栅栏时间戳到 HW VSYNC 信号的时间以纳秒为单位进行测量。

### **VSYNC 和 SF_VSYNC 偏移**

  
VSYNC_EVENT_PHASE_OFFSET_NS 和 SF_VSYNC_EVENT_PHASE_OFFSET_NS
根据高负载使用情形进行了保守设置，例如在窗口过渡期间进行部分 GPU 合成或 Chrome 滚动显示包含动画的网页。这些偏移允许较长的应用渲染时间和较长的
GPU 合成时间。

超过一两毫秒的延迟时间是非常明显的。为了最大限度地缩短延迟时间而不显著增加错误计数，请集成彻底的自动化错误测试。

注意：VSYNC 和 SF_VSYNC 偏移同样在设备的 BoardConfig.mk 文件中配置。两个设置都是 HW_VSYNC_0
之后以纳秒为单位的偏移，默认值为零（如未设置的话），也可以为负值。

