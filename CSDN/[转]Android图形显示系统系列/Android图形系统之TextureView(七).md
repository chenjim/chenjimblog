**原文** [Android图形系统之TextureView(七)](https://unbroken.blog.csdn.net/article/details/122407013)  

**TextureView 类是一个结合了 View 和 SurfaceTexture 的 View 对象。**

## **1.使用 OpenGL ES 呈现**

  
TextureView 对象会对 SurfaceTexture 进行包装，从而响应回调以及获取新的缓冲区。在 TextureView
获取新的缓冲区时，TextureView 会发出 View 失效请求，并使用最新缓冲区的内容作为数据源进行绘图，根据 View
状态的指示，以相应的方式在相应的位置进行呈现。

OpenGL ES (GLES) 可以将 SurfaceTexture 传递到 EGL 创建调用，从而在 TextureView
上呈现内容，但这样会引发问题。当 GLES 在 TextureView 上呈现内容时，BufferQueue
生产方和使用方位于同一线程中，这可能导致缓冲区交换调用暂停或失败。例如，如果生产方以快速连续的方式从界面线程提交多个缓冲区，则 EGL
缓冲区交换调用需要使一个缓冲区从 BufferQueue
出列。不过，由于使用方和生产方位于同一线程中，因此不存在任何可用的缓冲区，而且交换调用会挂起或失败。

为了确保缓冲区交换不会停止，BufferQueue 始终需要有一个可用的缓冲区能够出列。为了实现这一点，BufferQueue
在新缓冲区加入队列时舍弃之前加入队列的缓冲区的内容，并对最小缓冲区计数和最大缓冲区计数施加限制，以防使用方一次性消耗所有缓冲区。

## **2.选择 SurfaceView 或 TextureView**

  
注意：在 API 24 及更高版本中，建议实现 SurfaceView 而不是 TextureView。  
SurfaceView 和 TextureView 扮演的角色类似，且都是视图层次结构的组成部分。不过，SurfaceView 和 TextureView
拥有截然不同的实现。SurfaceView 采用与其他 View 相同的参数，但 SurfaceView 内容在呈现时是透明的。

与 SurfaceView 相比，TextureView 具有更出色的 Alpha
版和旋转处理能力，但在视频上以分层方式合成界面元素时，SurfaceView 具有性能方面的优势。当客户端使用 SurfaceView
呈现内容时，SurfaceView 会为客户端提供单独的合成层。如果设备支持，SurfaceFlinger 会将单独的层合成为硬件叠加层。当客户端使用
TextureView 呈现内容时，界面工具包会使用 GPU 将 TextureView 的内容合成到视图层次结构中。对内容进行的更新可能会导致其他
View 元素重绘，例如，在其他 View 被置于 TextureView 顶部时。View 呈现完成后，SurfaceFlinger
会合成应用界面层和所有其他层，以便每个可见像素合成两次。

注意：受 DRM 保护的视频只能在叠加平面上呈现。支持受保护内容的视频播放器必须使用 SurfaceView 实现。

