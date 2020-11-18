**原文** [Android图形系统之Surface与SurfaceHolder关系(四)](https://unbroken.blog.csdn.net/article/details/122406620)  

  
Surface 对象使应用能够渲染要在屏幕上显示的图像。通过 SurfaceHolder 接口，应用可以编辑和控制 Surface。

## **1.Surface**

  
Surface 是一个接口，供生产方与消耗方交换缓冲区。

用于显示 Surface 的 BufferQueue 通常配置为三重缓冲。缓冲区是按需分配的，因此，如果生产方足够缓慢地生成缓冲区（例如在 60 fps
的显示屏上以 30 fps 的速度进行缓冲），队列中可能只有两个分配的缓冲区。按需分配缓冲区有助于最大限度地减少内存消耗。您可以在 dumpsys
SurfaceFlinger 的输出中看到每个层级关联的缓冲区的摘要。

大多数客户端使用 OpenGL ES 或 Vulkan 渲染到 Surface 上。但是，有些客户端使用画布渲染到 Surface 上。

## **2.画布渲染**

  
画布实现由 Skia 图形库提供。如果您要绘制一个矩形，可以调用 Canvas
API，它会在缓冲区中适当地设置字节。为了确保两个客户端不会同时更新某个缓冲区，或者在该缓冲区正在被显示时写入该缓冲区，请锁定该缓冲区以进行访问。您可以使用以下命令处理画布锁：

lockCanvas() 可锁定缓冲区以在 CPU 上渲染，并返回用于绘图的画布。  
unlockCanvasAndPost() 可解锁缓冲区并将其发送到合成器。  
lockHardwareCanvas() 可锁定缓冲区以在 GPU 上渲染，并返回用于绘图的画布。  
注意：当应用通过 lockCanvas() 锁定 Surface 时，所获得的画布一概不会获得硬件加速。  
注意：如果您曾调用过 lockCanvas()，则无法使用 GLES 在 Surface 上绘图或从视频解码器向其发送帧。lockCanvas() 会将
CPU 渲染程序连接到 BufferQueue 的生产方，直到 Surface 被销毁时才会断开连接。与大多数生产方（如 GLES 或
Vulkan）不同，基于画布的 CPU 渲染程序无法在断开连接后重新连接到 Surface。  
生产方首次从 BufferQueue
请求某个缓冲区时，该缓冲区将被分配并初始化为零。必须进行初始化，以避免意外地在进程之间共享数据。但是，如果您重复使用缓冲区，以前的内容仍会存在。如果您反复调用
lockCanvas() 和 unlockCanvasAndPost() 而不绘制任何内容，则生产方会在先前渲染的帧之间循环。

Surface 锁定/解锁代码会保留对先前渲染的缓冲区的引用。如果在锁定 Surface
时指定了脏区域，那么它将从以前的缓冲区复制非脏像素。SurfaceFlinger 或 HWC
通常会处理缓冲区；但是由于我们只需从缓冲区中读取内容，因此无需等待独占访问权。

## **3.SurfaceHolder**

  
SurfaceHolder 是系统用于与应用共享 Surface 所有权的接口。与 Surface 配合使用的一些客户端需要
SurfaceHolder，因为用于获取和设置 Surface 参数的 API 是通过 SurfaceHolder 实现的。一个 SurfaceView
包含一个 SurfaceHolder。

与 View 交互的大多数组件都涉及到 SurfaceHolder。一些其他 API（如 MediaCodec）将在 Surface 本身上运行。

