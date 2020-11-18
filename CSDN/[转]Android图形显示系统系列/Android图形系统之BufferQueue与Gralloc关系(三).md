**原文** [Android图形系统之BufferQueue与Gralloc关系(三)](https://unbroken.blog.csdn.net/article/details/122406416)  

##

BufferQueue
类将可生成图形数据缓冲区的组件（生产方）连接到接受数据以便进行显示或进一步处理的组件（使用方）。几乎所有在系统中移动图形数据缓冲区的内容都依赖于
BufferQueue。

Gralloc 内存分配器会进行缓冲区分配，并通过两个特定于供应商的 HIDL 接口来进行实现（请参阅
`hardware/interfaces/graphics/allocator/` 和
`hardware/interfaces/graphics/mapper/`）。`allocate()`
函数采用预期的参数（宽度、高度、像素格式）以及一组用法标志。

## 1.BufferQueue 生产方和使用方

使用方创建并拥有 BufferQueue 数据结构，并且可存在于与其生产方不同的进程中。当生产方需要缓冲区时，它会通过调用
`dequeueBuffer()` 从 BufferQueue
请求一个可用的缓冲区，并指定缓冲区的宽度、高度、像素格式和用法标志。然后，生产方填充缓冲区并通过调用 `queueBuffer()`
将缓冲区返回到队列。接下来，使用方通过 `acquireBuffer()` 获取该缓冲区并使用该缓冲区的内容。当使用方操作完成后，它会通过调用
`releaseBuffer()` 将该缓冲区返回到队列。同步框架可控制缓冲区在 Android 图形管道中移动的方式。

BufferQueue 的一些特性（例如可以容纳的最大缓冲区数）由生产方和使用方联合决定。但是，BufferQueue
会根据需要分配缓冲区。除非特性发生变化，否则将会保留缓冲区；例如，如果生产方请求具有不同大小的缓冲区，系统会释放旧的缓冲区，并根据需要分配新的缓冲区。

BufferQueue 永远不会复制缓冲区内容，因为移动如此多的数据是非常低效的操作。相反，缓冲区始终通过句柄进行传递。

## 2.使用 Systrace 跟踪 BufferQueue

如需了解图形缓冲区如何移动，请使用 Systrace，它是一款用于记录短期内的设备活动的工具。系统级图形代码经过充分插桩，很多相关的应用框架代码也是如此。

如需使用 Systrace，请启用 `gfx`、`view` 和 `sched` 标记。BufferQueue 对象显示在跟踪记录中。例如，如果您在
Grafika 的播放视频 (SurfaceView)正在运行时获取跟踪记录，标有 SurfaceView
的行会告诉您在任何给定时间内加入队列的缓冲区数量。

当应用处于活动状态时，该值会递增，这会触发 MediaCodec 解码器渲染帧。当 SurfaceFlinger 正在工作和使用缓冲区时，该值会递减。当以
30fps 的帧率显示视频时，队列的值从 0 变为 1，因为大约 60fps 的显示速度可以轻松跟上来源的帧率。SurfaceFlinger
仅在有工作要执行时才会被唤醒，而不是每秒将其唤醒 60 次。系统会尝试避免工作，并且如果屏幕没有任何更新，将停用 VSYNC。

如果您切换到 Grafika 的播放视频 (SurfaceView)并获取新的跟踪记录，您将看到一个标有 `com.android.grafika` /
`com.android.grafika.PlayMovieActivity` 的行。这是主界面层，它是另一个 BufferQueue。由于
TextureView 渲染到界面层（而不是单独的层），因此此处将显示所有视频驱动的更新。

## 3.Gralloc

Gralloc 分配器 HAL `hardware/libhardware/include/hardware/gralloc.h`
通过用法标志执行缓冲区分配。用法标志包括以下属性：

  * 从软件 (CPU) 访问内存的频率
  * 从硬件 (GPU) 访问内存的频率
  * 是否将内存用作 OpenGL ES (GLES) 纹理
  * 视频编码器是否会使用内存

例如，如果生产方的缓冲区格式指定 `RGBA_8888` 像素，并且生产方指明将从软件访问缓冲区（这意味着应用将在 CPU 上触摸像素），则 Gralloc
将按照 R-G-B-A 的顺序为每个像素创建 4 个字节的缓冲区。如果情况相反，生产方指明仅从硬件访问其缓冲区且缓冲区作为 GLES 纹理，Gralloc
可以执行 GLES 驱动程序所需的任何操作（比如 BGRA 排序、非线性搅和布局和替代颜色格式等）。允许硬件使用其首选格式可以提高性能。

某些值在特定平台上无法组合。例如，视频编码器标记可能需要 YUV 像素，因此将无法添加软件访问权限并指定 `RGBA_8888`。

Gralloc 返回的句柄可以通过 Binder 在进程之间进行传递。

## **4.受保护的缓冲区**

Gralloc 使用标记 `GRALLOC_USAGE_PROTECTED` 允许仅通过受硬件保护的路径显示图形缓冲区。这些叠加平面是显示 DRM
内容的唯一途径（SurfaceFlinger 或 OpenGL ES 驱动程序无法访问受 DRM 保护的缓冲区）。

受 DRM 保护的视频只能在叠加平面上呈现。支持受保护内容的视频播放器必须使用 SurfaceView
实现。在不受保护的硬件上运行的软件无法读取或写入缓冲区；受硬件保护的路径必须显示在硬件混合渲染器叠加层上（也就是说，如果硬件混合渲染器切换到 OpenGL
ES 合成，受保护的视频将从屏幕中消失）。

