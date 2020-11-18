**原文** [Android图形系统之HWC与GPU混合渲染(九)](https://unbroken.blog.csdn.net/article/details/122409765)  

### 1.HWC

硬件混合渲染器 (HWC) HAL 用于确定通过可用硬件来合成缓冲区的最有效方法。作为 HAL，其实现是特定于设备的，而且通常由显示硬件原始设备制造商
(OEM) 完成。

当您考虑使用叠加平面时，很容易发现这种方法的好处，它会在显示硬件（而不是 GPU）中合成多个缓冲区。例如，假设有一部普通 Android
手机，其屏幕方向为纵向，状态栏在顶部，导航栏在底部，其他区域显示应用内容。每个层的内容都在单独的缓冲区中。您可以使用以下任一方法处理合成：

  * 将应用内容渲染到暂存缓冲区中，然后在其上渲染状态栏，再在其上渲染导航栏，最后将暂存缓冲区传送到显示硬件。
  * 将三个缓冲区全部传送到显示硬件，并指示它从不同的缓冲区读取屏幕不同部分的数据。

后一种方法可以显著提高效率。

显示处理器功能差异很大。叠加层的数量（无论层是否可以旋转或混合）以及对定位和重叠的限制很难通过 API 表达。为了适应这些选项，HWC 会执行以下计算：

  1. **SurfaceFlinger 向 HWC 提供一个完整的层列表，并询问“您希望如何处理这些层？”**
  2. **HWC 的响应方式是将每个层标记为设备或客户端合成。**
  3. **SurfaceFlinger 会处理所有客户端，将输出缓冲区传送到 HWC，并让 HWC 处理其余部分。**

由于硬件供应商可以定制决策代码，因此可以在每台设备上实现最佳性能。

当屏幕上的内容没有变化时，叠加平面的效率可能会低于 GL 合成。当叠加层内容具有透明像素且叠加层混合在一起时，尤其如此。在此类情况下，HWC
可以为部分或全部层请求 GLES 合成，并保留合成的缓冲区。如果 SurfaceFlinger 要求合成同一组缓冲区，HWC
可以显示先前合成的暂存缓冲区。这可以延长闲置设备的电池续航时间。

Android 设备通常支持 4 个叠加平面。尝试合成的层数多于叠加层数会导致系统对其中一些层使用 GLES
合成，这意味着应用使用的层数会对能耗和性能产生重大影响。

硬件混合渲染器 (HWC) HAL 用于合成从 SurfaceFlinger 接收的图层，从而减少 OpenGL ES (GLES) 和 GPU
执行的合成量。

HWC 可以抽象出叠加层和 2D 位块传送器等对象来合成 Surface，并与专门的窗口合成硬件进行通信以合成窗口。使用 HWC 来合成窗口，而不是让
SurfaceFlinger 与 GPU 进行合成。大多数 GPU 都未针对合成进行过优化，当 GPU 合成来自 SurfaceFlinger
的图层时，应用就无法使用 GPU 进行自我渲染。

HWC 实现应该支持：

至少 4 个叠加层：  
状态栏  
系统栏  
应用  
壁纸/背景  
大于屏幕的图层（例如壁纸）  
同时预乘的每像素 Alpha 混合和每平面 Alpha 混合  
受保护视频播放的硬件路径  
RGBA 打包顺序、YUV 格式以及平铺、重排和步幅属性  
要实现 HWC，请执行以下操作：

实现非运行的 HWC，并将所有合成工作发送到 GLES。  
实现一种算法，以逐步将合成委托给 HWC。例如，仅将前 3 个或前 4 个 Surface 委托给 HWC 的叠加硬件。  
优化 HWC。这可能包括以下操作：  
选择可以最大限度提高从 GPU 移除的负载的 Surface，并将其发送到 HWC。  
检测屏幕是否正在更新。如果不是，则将合成委托给 GLES 而不是 HWC，以节省电量。当屏幕再次更新时，继续将合成分载到 HWC。  
为常见用例做准备，例如：  
主屏幕，包括状态栏、系统栏、应用窗口和动态壁纸  
纵向和横向模式下的全屏游戏  
带有字幕和播放控件的全屏视频  
受保护的视频播放  
分屏多窗口  
注意：用例应针对常规可预测的用途而不是边缘用例，以便最大限度地提高优化益处。

  
**HWC 基元**  
HWC 提供了两个基元（图层和屏幕）来表示合成工作及其与屏幕硬件的交互。此外，HWC 还提供对 VSYNC 的控制，以及对 SurfaceFlinger
的回调，用于通知它何时发生 VSYNC 事件。

**HIDL 接口**  
Android 8.0 及更高版本使用一个名为混合渲染器 HAL 的 HIDL 接口，用于在 HWC 和 SurfaceFlinger 之间绑定
IPC。混合渲染器 HAL 取代了旧版 hwcomposer2.h 接口。如果供应商提供 HWC 的混合渲染器 HAL 实现，则混合渲染器 HAL
将直接接受来自 SurfaceFlinger 的 HIDL 调用。如果供应商提供 HWC 的旧版实现，混合渲染器 HAL 将从 hwcomposer2.h
加载函数指针，从而将 HIDL 调用转发到函数指针调用中。

HWC 提供相应函数来确定给定屏幕的属性；在不同的屏幕配置（例如 4k 或 1080p 分辨率）和颜色模式（例如原生颜色或真正的
sRGB）之间切换；以及打开、关闭屏幕或将其切换到低功率模式（如果支持）。

**函数指针**  
如果供应商直接实现混合渲染器 HAL，则 SurfaceFlinger 会通过 HIDL IPC 调用其函数。例如，要创建图层，SurfaceFlinger
会在混合渲染器 HAL 上调用 createLayer()。

如果供应商实现 hwcomposer2.h 接口，则混合渲染器 HAL 会调用 hwcomposer2.h 函数指针。在 hwcomposer2.h
注释中，HWC 接口函数由 lowerCamelCase 名称引用，这些名称并未作为命名的字段存在于接口中。几乎每个函数都是通过使用
hwc2_device_t 提供的 getFunction 请求函数指针来进行加载。例如，函数 createLayer 是一个
HWC2_PFN_CREATE_LAYER 类型的函数指针，当枚举值 HWC2_FUNCTION_CREATE_LAYER 传递到 getFunction
中时便会返回该指针。

有关混合渲染器 HAL 函数和 HWC 直通式函数的详细文档，请参阅 composer。有关 HWC 函数指针的详细文档，请参阅
hwcomposer2.h。

**图层和屏幕句柄**  
图层和屏幕由 HWC 生成的句柄操纵。这些句柄对 SurfaceFlinger 而言是不透明的。

当 SurfaceFlinger 要创建新图层时，它会调用 createLayer，该函数会针对直接实现返回 Layer 类型，或者针对直通式实现返回
hwc2_layer_t 类型。当 SurfaceFlinger 要修改该图层的属性时，它会将该 hwc2_layer_t
值以及进行修改所需的任何其他信息传递给相应的修改函数。hwc2_layer_t 类型的大小足以容纳一个指针或一个索引。

物理屏幕是通过热插拔创建的。在热插拔物理屏幕时，HWC 会创建一个句柄并通过热插拔回调将该句柄传递给 SurfaceFlinger。虚拟屏幕是由通过调用
createVirtualDisplay() 来请求屏幕的 SurfaceFlinger 创建的。如果 HWC
支持虚拟屏幕合成，则会返回一个句柄。然后，SurfaceFlinger 将屏幕的合成工作委托给 HWC。如果 HWC 不支持虚拟屏幕合成，则
SurfaceFlinger 会创建句柄并合成屏幕。

**屏幕合成操作**  
如果 SurfaceFlinger 具有可合成的新内容，则会唤醒，且每个 VSYNC
唤醒一次。该新内容可以是来自应用的新图像缓冲区，也可以是一个或多个图层的属性更改。当 SurfaceFlinger 唤醒时，它将执行以下步骤：

处理事务（如果存在）。  
如果存在新的图形缓冲区，则将其锁定。  
如果步骤 1 或 2 导致显示内容更改，则执行新的合成。  
为了执行新的合成，SurfaceFlinger 会创建和销毁图层或修改图层状态（如适用）。它还会使用 setLayerBuffer 或
setLayerColor 等调用，用图层的当前内容来更新图层。更新所有图层之后，SurfaceFlinger 会调用
validateDisplay，以告诉 HWC 检查各个图层的状态，并确定如何进行合成。尽管在某些情况下，SurfaceFlinger 会通过 GPU
回退来合成图层，但在默认情况下，SurfaceFlinger 会尝试配置每个图层，以使相应图层由 HWC 进行合成。

在调用 validateDisplay 之后，SurfaceFlinger 会调用 getChangedCompositionTypes，以查看 HWC
是否需要在执行合成之前更改任何图层的合成类型。要接受这些更改，SurfaceFlinger 需要调用 acceptDisplayChanges。

如果已将任何图层标记为进行 SurfaceFlinger 合成，SurfaceFlinger
会将这些图层合成到目标缓冲区中。然后，SurfaceFlinger 通过调用 setClientTarget
将该缓冲区提供给屏幕，以便缓冲区可以在屏幕上显示，或者进一步与未标记为进行 SurfaceFlinger 合成的图层合成。如果没有将任何图层标记为进行
SurfaceFlinger 合成，则 SurfaceFlinger 会跳过合成步骤。

最后，SurfaceFlinger 会调用 presentDisplay，以告知 HWC 完成合成过程并显示最终结果。

**多个屏幕**  
Android 10 支持多个物理屏幕。在设计要在 Android 7.0 及更高版本上使用的 HWC 实现时，还存在一些 HWC 定义中并不存在的限制：

假设只有一个内部屏幕。内部屏幕是初始热插拔在启动期间报告的屏幕。内部屏幕在热插拔后无法断开连接。  
在设备的正常操作期间，除了内部屏幕之外，还可以热插拔任意数量的外部屏幕。该框架假设，在热插拔第一个内部屏幕之后，所有热插拔操作都是在外部屏幕上进行，因此如果添加了更多内部屏幕，它们将被错误地归类为
Display.TYPE_HDMI，而不是 Display.TYPE_BUILT_IN。  
尽管上述 SurfaceFlinger 操作按屏幕执行，但会对所有活动的屏幕依序执行这些操作，即使只更新了一个屏幕的内容也不例外。

例如，如果更新了外部屏幕，则顺序为：

  
// In Android 9 and lower:

// Update state for internal display  
// Update state for external display  
validateDisplay()  
validateDisplay()  
presentDisplay()  
presentDisplay()

// In Android 10 and higher:

// Update state for internal display  
// Update state for external display  
validateInternal()  
presentInternal()  
validateExternal()  
presentExternal()

  
**虚拟屏幕合成**  
虚拟屏幕合成与外部屏幕合成类似。虚拟屏幕合成与物理屏幕合成之间的区别在于，虚拟屏幕将输出发送到 Gralloc 缓冲区，而不是发送到屏幕。硬件混合渲染器
(HWC) 将输出内容写入缓冲区，提供完成栅栏信号，并将缓冲区发送给使用方（例如视频编码器、GPU、CPU 等）。如果显示通道写入内存，虚拟屏幕便可使用
2D/位块传送器或叠加层。

**模式**  
**在 SurfaceFlinger 调用 validateDisplay() HWC 方法之后，每个帧处于以下三种模式之一：**

**GLES - GPU 合成所有图层，直接写入输出缓冲区。HWC 不参与合成。  
MIXED - GPU 将一些图层合成到帧缓冲区，由 HWC 合成帧缓冲区和剩余的图层，直接写入输出缓冲区。  
HWC - HWC 合成所有图层并直接写入输出缓冲区。**

  
**输出格式**  
虚拟屏幕的缓冲区输出格式取决于其模式：

GLES 模式 - EGL 驱动程序在 dequeueBuffer() 中设置输出缓冲区格式，通常为
RGBA_8888。使用方必须能够接受该驱动程序设置的输出格式，否则便无法读取缓冲区。  
MIXED 和 HWC 模式 - 如果使用方需要 CPU 访问权限，则由使用方设置格式。否则，格式将为
IMPLEMENTATION_DEFINED，Gralloc 会根据使用标记设置最佳格式。例如，如果使用方是视频编码器，则 Gralloc 会设置
YCbCr 格式，而 HWC 可以高效地写入该格式。  
注意：Android 10 取消了有关 eglSwapBuffers() 在渲染开始后将缓冲区移出队列的要求。缓冲区可能会立即退出队列。

  
**同步栅栏**  
同步栅栏是 Android 图形系统的关键部分。栅栏允许 CPU 工作与并行的 GPU 工作相互独立进行，仅在存在真正的依赖关系时才会阻塞。

例如，当应用提交在 GPU 上生成的缓冲区时，它还将提交一个同步栅栏对象。当 GPU 完成写入缓冲区的操作时，该栅栏会变为有信号状态。

HWC 要求 GPU 在显示缓冲区之前完成写入缓冲区的操作。同步栅栏通过包含缓冲区的图形管道传递，并在缓冲区写入完成时发出信号。在显示缓冲区之前，HWC
会检查同步栅栏是否已发出信号，如果有，则显示缓冲区。

如需详细了解同步栅栏，请参阅硬件混合渲染器集成。

