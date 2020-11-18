**原文** [Android图形系统之SurfaceTexture(六)](https://unbroken.blog.csdn.net/article/details/122406810)  

## **1.SurfaceTexture**

  
SurfaceTexture 是 Surface 和 OpenGL ES (GLES) 纹理的组合。SurfaceTexture 实例用于提供输出到
GLES 纹理的接口。

SurfaceTexture 包含一个以应用为使用方的 BufferQueue 实例。当生产方将新的缓冲区排入队列时，onFrameAvailable()
回调会通知应用。然后，应用调用 updateTexImage()，这会释放先前占用的缓冲区，从队列中获取新缓冲区并执行 EGL 调用，从而使 GLES
可将此缓冲区作为外部纹理使用。

## **2.外部 GLES 纹理**

  
外部 GLES 纹理 (GL_TEXTURE_EXTERNAL_OES) 与传统 GLES 纹理 (GL_TEXTURE_2D) 的区别如下：

外部纹理直接在从 BufferQueue 接收的数据中渲染纹理多边形。  
外部纹理渲染程序的配置与传统的 GLES 纹理渲染程序不同。  
外部纹理不一定可以执行所有传统的 GLES 纹理活动。  
外部纹理的主要优势是它们能够直接从 BufferQueue 数据进行渲染。SurfaceTexture 实例在为外部纹理创建 BufferQueue
实例时将使用方用法标志设置为 GRALLOC_USAGE_HW_TEXTURE，以确保 GLES 可以识别该缓冲区中的数据。

由于 SurfaceTexture 实例会与 EGL 上下文交互，所以当拥有纹理的 EGL
上下文当前在发起调用的线程上时，应用只能调用其自己的方法。如需了解详情，请参阅 SurfaceTexture 类文档。

## **3.时间戳和转换**

  
SurfaceTexture 实例包含用于检索时间戳的 getTimeStamp() 方法和用于检索转换矩阵的 getTransformMatrix()
方法。调用 updateTexImage() 可设置时间戳和转换矩阵。BufferQueue 传递的每个缓冲区都包含转换参数和时间戳。

转换参数有利于提升效率。在某些情况下，源数据可能以错误的方向传递给使用方。
我们可以按照数据的方向发送数据并使用转换对其进行更正，而不是在将数据发送到使用方之前对其进行旋转。在使用数据时，转换矩阵可以与其他转换合并，从而最大限度降低开销。

时间戳对于与时间相关的缓冲区来源非常有用。例如，当 setPreviewTexture()
将生产方接口连接到相机的输出端时，可以使用相机的帧来创建视频。每一帧都需要根据截取帧的时间（而不是应用收到帧的时间）来设置演示时间戳。相机代码可设置随缓冲区提供的时间戳，从而获得一系列更一致的时间戳。

## **4.Grafika 的连续拍摄**

  
Grafika 的连续拍摄涉及从设备相机录制帧并在屏幕上显示这些帧。 要录制帧，请使用 MediaCodec 类的 createInputSurface()
方法创建 Surface，并将该 Surface 传递给相机。要显示帧，请创建 SurfaceView 的实例并将 Surface 传递给
setPreviewDisplay()。请注意，在录制帧的同时进行显示是一个更复杂的过程。

在录制视频时，连续拍摄 Activity 会显示相机录制的视频。在这种情况下，已编码的视频将写入内存中的环形缓冲区，该缓冲区中的内容可随时保存到磁盘。

此流程涉及三个缓冲区队列：

App - 该应用使用 SurfaceTexture 实例从相机接收帧，并将其转换为外部 GLES 纹理。  
SurfaceFlinger - 应用声明用来显示帧的 SurfaceView 实例。  
MediaServer - 您可以使用输入 Surface 配置 MediaCodec 编码器，以创建视频。  
在上图中，箭头指示相机的数据传输路径。 BufferQueue 实例则用颜色标示（生产方为青色，使用方为绿色）。

Grafika 连续拍摄 Activity

![](https://img-blog.csdnimg.cn/159f715e817b470e8cbe094b72386ab0.png?x-oss-
process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oWi5oWi55qE54eD54On,size_17,color_FFFFFF,t_70,g_se,x_16)  
图 1. Grafika 的连续拍摄 Activity

已编码的 H.264 视频在应用进程中进入 RAM 中的环形缓冲区。 当用户按下录像按钮时，MediaMuxer 类会将已编码的视频写入到磁盘上的 MP4
文件中。

所有 BufferQueue 实例都在应用中通过单个 EGL 上下文处理，而 GLES
操作在界面线程上执行。对已编码数据的处理（管理环形缓冲区并将其写入磁盘）是在单独的线程上完成的。

警告：如果视频编码器锁定并阻止将缓冲区移出队列，则应用将停止响应。  
注意：不推荐在界面线程上执行 SurfaceView 渲染，但为了简单起见，本案例研究中依然使用了这种做法。更复杂的渲染应使用专用线程来隔离 GLES
上下文，并最大限度地减少对另一应用的界面渲染的干扰。  
使用 SurfaceView 类时，surfaceCreated() 回调会为屏幕和视频编码器创建 EGLContext 和 EGLSurface
实例。当新帧到达时，SurfaceTexture 将执行四项活动：  
获取框架。  
使框架能够作为 GLES 纹理提供。  
使用 GLES 命令渲染帧。  
转发 EGLSurface 的每个实例的转换和时间戳。  
然后，编码器线程从 MediaCodec 提取编码的输出内容，并将其存储在内存中。

## **5.安全纹理视频播放**

  
Android 支持对受保护的视频内容进行 GPU 后处理。这使得应用可以将 GPU
用于复杂的非线性视频效果（例如扭曲），将受保护的视频内容映射到纹理，以用于常规图形场景（例如，使用 GLES）和虚拟现实 (VR)。

安全纹理视频播放

![](https://img-blog.csdnimg.cn/4065bc35617c43269f15dd95a993c2a7.png?x-oss-
process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5oWi5oWi55qE54eD54On,size_18,color_FFFFFF,t_70,g_se,x_16)  
图 2. 安全纹理视频播放

使用以下两个扩展程序实现支持：

EGL 扩展程序 - (EGL_EXT_protected_content) 允许创建受保护的 GL 上下文和
Surface，它们都可以对受保护的内容进行操作。  
GLES 扩展程序 - (GL_EXT_protected_textures) 允许将纹理标记为受保护，以便它们可以用作帧缓冲区纹理附件。  
即使窗口 Surface 没有加入 SurfaceFlinger 的队列，Android 也支持 SurfaceTexture 和 ACodec
(libstagefright.so) 发送受保护的内容，并且提供受保护的视频 Surface 以在受保护的上下文中使用。该操作通过在受保护的上下文（由
ACodec 验证）中创建的 Surface 上设置受保护的使用方位 (GRALLOC_USAGE_PROTECTED) 来完成。

安全纹理视频播放为在 OpenGL ES 环境中进行强大的 DRM 实现奠定了基础。如果没有诸如 Widevine 级别 1 这样强大的 DRM
实现，许多内容提供商将不允许在 OpenGL ES 环境中渲染其高价值内容，从而阻止重要的 VR 使用情形（例如在 VR 中观看受 DRM 保护的内容）。

AOSP 包括用于安全纹理视频播放的框架代码。驱动程序支持取决于原始设备制造商 (OEM)。设备实现人员必须实现
EGL_EXT_protected_content 和 GL_EXT_protected_textures
extensions。使用您自己的编解码器库（替代 libstagefright）时，请注意，只要使用方使用位包含
GRALLOC_USAGE_PROTECTED，/frameworks/av/media/libstagefright/SurfaceUtils.cpp
中的更改便会允许将标记为 GRALLOC_USAGE_PROTECTED 的缓冲区发送到 ANativeWindow，即便 ANativeWindow
没有直接加入窗口编写器的队列也可以。有关如何实现扩展程序的详细文档，请参阅 Khronos 注册表（EGL_EXT_protected_content 和
GL_EXT_protected_textures）。

注意：设备实现人员可能需要进行硬件更改，才能确保映射到 GPU 的受保护内存仍受保护，且不可由未受保护的代码读取。

