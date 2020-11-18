**原文** [Android图形系统之SurfaceFlinger与WindowManager(八)](https://unbroken.blog.csdn.net/article/details/122407138)  

SurfaceFlinger 接受缓冲区，对它们进行合成，然后发送到屏幕。WindowManager 为 SurfaceFlinger
提供缓冲区和窗口元数据，而 SurfaceFlinger 可使用这些信息将 Surface 合成到屏幕。

## **1.SurfaceFlinger**

  
SurfaceFlinger 可通过两种方式接受缓冲区：通过 BufferQueue 和 SurfaceControl，或通过
ASurfaceControl。

SurfaceFlinger 接受缓冲区的一种方式是通过 BufferQueue 和 SurfaceControl。当应用进入前台时，它会从
WindowManager 请求缓冲区。然后，WindowManager 会从 SurfaceFlinger 请求层。层是 surface（包含
BufferQueue）和 SurfaceControl（包含显示框架等层元数据）的组合。SurfaceFlinger 创建层并将其发送至
WindowManager。然后，WindowManager 将 Surface 发送至应用，但会保留 SurfaceControl
来操控应用在屏幕上的外观。

Android 10 新增了 ASurfaceControl，这是 SurfaceFlinger 接受缓冲区的另一种方式。ASurfaceControl 将
Surface 和 SurfaceControl 组合到一个事务包中，该包会被发送至 SurfaceFlinger。ASurfaceControl
与层相关联，应用可通过 ASurfaceTransactions 更新该层。然后，应用可通过回调（用于传递包含锁定时间、获取时间等信息的
ASurfaceTransactionStats）获取有关 ASurfaceTransactions 的信息。

下表包含有关 ASurfaceControl 及其相关组件的更多详细信息。

组件 说明  
ASurfaceControl 对 SurfaceControl 进行封装并使应用能够创建与屏幕上的各层相对应的 SurfaceControl。

可作为 ANativeWindow 的一个子级或者另一个 ASurfaceControl 的子级创建。  
ASurfaceTransaction 对事务进行包装，以使客户端能够修改层的描述性属性（比如几何图形），并将经过更新的缓冲区发送至
SurfaceFlinger。  
ASurfaceTransactionStats 通过预先注册的回调将有关已显示事务的信息（比如锁定时间、获取时间和上一个释放栅栏）发送至应用。  
虽然应用可以随时提交缓冲区，但 SurfaceFlinger
仅能在屏幕处于两次刷新之间时唤醒，以接受缓冲区，这会因设备而异。这样可以最大限度地减少内存使用量，并避免屏幕上出现可见的撕裂现象（如果显示内容在刷新期间更新，则会出现此现象）。

在屏幕处于两次刷新之间时，屏幕会向 SurfaceFlinger 发送 VSYNC 信号。VSYNC 信号表明可对屏幕进行刷新而不会产生撕裂。当
SurfaceFlinger 接收到 VSYNC 信号后，SurfaceFlinger 会遍历其层列表，以查找新的缓冲区。如果 SurfaceFlinger
找到新的缓冲区，SurfaceFlinger 会获取缓冲区；否则，SurfaceFlinger
会继续使用上一次获取的那个缓冲区。SurfaceFlinger 必须始终显示内容，因此它会保留一个缓冲区。如果在某个层上没有提交缓冲区，该层会被忽略。

SurfaceFlinger 在收集可见层的所有缓冲区之后，便会询问硬件混合渲染器 (HWC) 应如何进行合成。如果 HWC
将层合成类型标记为客户端合成，则 SurfaceFlinger 将合成这些层。然后，SurfaceFlinger 会将输出缓冲区传递给 HWC。

## **2.WindowManager**

  
WindowManager 会控制窗口对象，它们是用于容纳视图对象的容器。窗口对象始终由 Surface 对象提供支持。WindowManager
会监督生命周期、输入和聚焦事件、屏幕方向、转换、动画、位置、变形、Z 轴顺序以及窗口的许多其他方面。WindowManager 会将所有窗口元数据发送到
SurfaceFlinger，以便 SurfaceFlinger 可以使用这些数据在屏幕上合成 Surface。

**预旋转**  
许多硬件叠加层不支持旋转（即使支持，也需要耗用处理能力）；解决方案就是在缓冲区到达 SurfaceFlinger 之前对其进行转换。Android 在
ANativeWindow 中支持查询提示 (NATIVE_WINDOW_TRANSFORM_HINT)，以表示最有可能由 SurfaceFlinger
应用于缓冲区的转换。GL 驱动程序可以使用此提示在缓冲区到达 SurfaceFlinger 之前对其进行预转换，以便在缓冲区到达时能对其进行正确的转换。

例如，当接收到旋转 90 度的提示时，会生成一个矩阵并将其应用于缓冲区，以防止其从页面末尾运行。为了节省电量，请进行此预旋转。如需了解详情，请参阅
system/core/include/system/window.h 中定义的 ANativeWindow 接口。

