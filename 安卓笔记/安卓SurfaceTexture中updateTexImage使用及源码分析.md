
[TOC]

>本文首发地址 <https://h89.cn/archives/140.html>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>

## 引言

在Android系统中，`SurfaceTexture` 是一个特殊的类，它将来自硬件纹理缓冲区（如相机预览流或视频解码输出）的图像数据转换为 OpenGL ES 可以直接使用的纹理。`updateTexImage()` 方法是 `SurfaceTexture` 类的核心方法之一，此方法的主要作用是从 SurfaceTexture 内部持有的图像缓冲区中取出最新一帧，并将其内容复制到与 SurfaceTexture 关联的 OpenGL 纹理上。这对于实时图形渲染、视频播放以及从相机捕获并实时处理图像等场景至关重要。

## updateTexImage 简单使用
下面是一个简化的使用 SurfaceTexture 与 GLSurfaceView 实现渲染的基本流程：  
```java
// 初始化 SurfaceTexture
SurfaceTexture surfaceTexture = new SurfaceTexture(GLES11Ext.GL_TEXTURE_EXTERNAL_OES);
surfaceTexture.setOnFrameAvailableListener(new SurfaceTexture.OnFrameAvailableListener() {
    @Override
    public void onFrameAvailable(SurfaceTexture st) {
        // 新帧到达时，通知主线程进行渲染
        // 在这里可以调用 requestRender() 或者通过 Handler 发送消息
    }
});

// 在GL线程中初始化纹理并绑定
fun initTexture(){
    int[] textureId = new int[1];
    GLES20.glGenTextures(1, textureId, 0);
    GLES20 glBindTexture(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, textureId[0]);

    // 设置纹理参数
    GLES20.glTexParameterf(GLES11Ext.GL_TEXTURE_EXTERNAL_OES,
            GL10.GL_TEXTURE_MIN_FILTER, GL10.GL_LINEAR);
    GLES20.glTexParameterf(GLES11Ext.GL_TEXTURE_EXTERNAL_OES,
            GL10.GL_TEXTURE_MAG_FILTER, GL10.GL_LINEAR);

    // 将SurfaceTexture关联到OpenGL纹理
    surfaceTexture.attachToGLContext(textureId[0]);
}

// 在onDrawFrame(GL10 gl)方法中更新和渲染纹理
@Override
public void onDrawFrame(GL10 unused) {
    // 检查是否有新帧可用
    if (surfaceTexture.isFrameAvailable()) {
        // 更新纹理数据
        surfaceTexture.updateTexImage();

        // 获取变换矩阵
        float[] transformMatrix = new float[16];
        surfaceTexture.getTransformMatrix(transformMatrix);

        // 使用该矩阵进行纹理坐标变换后渲染
        // ...
    }
}
// 注意：实际应用中需要确保在正确的线程上调用 updateTexImage() 和其他OpenGL函数
```

## SurfaceTexture 初始化相关源码分析  
android.graphics.SurfaceTexture 初始化  
```java
    // frameworks.base\graphics\java\android\graphics\SurfaceTexture.java
    public SurfaceTexture(int texName, boolean singleBufferMode) {
        mCreatorLooper = Looper.myLooper();
        mIsSingleBuffered = singleBufferMode;
        nativeInit(false, texName, singleBufferMode, new WeakReference<SurfaceTexture>(this));
    }
    public void updateTexImage() {
        nativeUpdateTexImage();
    }
```
android.graphics.SurfaceTexture JNI实现  
```cpp
// frameworks\base\core\jni\android_graphics_SurfaceTexture.cpp
static const JNINativeMethod gSurfaceTextureMethods[] = {
        {"nativeInit", "(ZIZLjava/lang/ref/WeakReference;)V", (void*)SurfaceTexture_init},
        {"nativeUpdateTexImage", "()V", (void*)SurfaceTexture_updateTexImage},
};

static void SurfaceTexture_updateTexImage(JNIEnv* env, jobject thiz) {
    // 最终调用了 C++ 对象 SurfaceTexture 的 updateTexImage
    status_t err = surfaceTexture->updateTexImage();
}

static void SurfaceTexture_init(JNIEnv* env, jobject thiz, jboolean isDetached,
        jint texName, jboolean singleBufferMode, jobject weakThiz)
{
    sp<IGraphicBufferProducer> producer;
    sp<IGraphicBufferConsumer> consumer;
    // 创建 生产者和消费者， createBufferQueue 详细源码见后文。
    BufferQueue::createBufferQueue(&producer, &consumer);

    if (singleBufferMode) {
        consumer->setMaxBufferCount(1);
    }

    sp<SurfaceTexture> surfaceTexture;
    if (isDetached) {
        // 一般情况只有 singleBufferMode 时，isDetached 为真
        surfaceTexture = new SurfaceTexture(consumer, GL_TEXTURE_EXTERNAL_OES,
                true, !singleBufferMode);
    } else {
        // 传入了 消费者  consumer
        surfaceTexture = new SurfaceTexture(consumer, texName,
                GL_TEXTURE_EXTERNAL_OES, true, !singleBufferMode);
    }

    // 设置名称 
    surfaceTexture->setName(String8::format("SurfaceTexture-%d-%d-%d",
            (isDetached ? 0 : texName),
            getpid(),
            createProcessUniqueId()));

    // 检查当前的 EGL 上下文是否受保护。
    consumer->setConsumerIsProtected(isProtectedContext());

    // 通过 fields.surfaceTexture 保存 surfaceTexture 到 android.graphics.SurfaceTexture 中 mSurfaceTexture
    SurfaceTexture_setSurfaceTexture(env, thiz, surfaceTexture);
    // 通过 fields.producer 保存 producer 到 android.graphics.SurfaceTexture 中 mProducer
    SurfaceTexture_setProducer(env, thiz, producer);


    sp<JNISurfaceTextureContext> ctx(new JNISurfaceTextureContext(env, weakThiz, clazz));
    // 回调 设置到 ConsumerBase 中
    surfaceTexture->setFrameAvailableListener(ctx);
    // 通过 fields.frameAvailableListener 保存 ctx 到 android.graphics.SurfaceTexture 中 mFrameAvailableListener
    SurfaceTexture_setFrameAvailableListener(env, thiz, ctx);
}

void JNISurfaceTextureContext::onFrameAvailable(const BufferItem& /* item */)
{
    // 回调到 android.graphics.SurfaceTexture 中 postEventFromNative
    // 进而 可以 通过 android.graphics.SurfaceTexture.setOnFrameAvailableListener 通知到应用
    env->CallStaticVoidMethod(mClazz, fields.postEvent, mWeakThiz);
}
```

SurfaceTexture 继承自 ConsumerBase, setFrameAvailableListener 如下  
```cpp
class ANDROID_API SurfaceTexture : public ConsumerBase {

}
void ConsumerBase::setFrameAvailableListener(const wp<FrameAvailableListener>& listener) {
    Mutex::Autolock lock(mFrameAvailableMutex);
    mFrameAvailableListener = listener;
}

```
SurfaceTexture_setFrameAvailableListener 如下 
```cpp
static void SurfaceTexture_setFrameAvailableListener(JNIEnv* env,
        jobject thiz, sp<SurfaceTexture::FrameAvailableListener> listener)
{
    // 拿到 android.graphics.SurfaceTexture 中 mFrameAvailableListener
    SurfaceTexture::FrameAvailableListener* const p =
        (SurfaceTexture::FrameAvailableListener*)env->GetLongField(thiz, fields.frameAvailableListener);
    ...
    // 更新到 android.graphics.SurfaceTexture 
    env->SetLongField(thiz, fields.frameAvailableListener, (jlong)listener.get());
}
```
谁会调用 JNISurfaceTextureContext::onFrameAvailable 呢
```cpp
void ConsumerBase::onFrameAvailable(const BufferItem& item) {

    sp<FrameAvailableListener> listener;
    {
        Mutex::Autolock lock(mFrameAvailableMutex);
        listener = mFrameAvailableListener.promote();
    }

    if (listener != nullptr) {
        // 也就是调用 JNISurfaceTextureContext::onFrameAvailable
        listener->onFrameAvailable(item);
    }
}
```
谁会调用 ConsumerBase::onFrameAvailable 呢  
```cpp
// frameworks\native\libs\nativedisplay\surfacetexture\SurfaceTexture.cpp
SurfaceTexture::SurfaceTexture(const sp<IGraphicBufferConsumer>& bq, uint32_t tex,
                               uint32_t texTarget, bool useFenceSync, bool isControlledByApp)
      : ConsumerBase(bq, isControlledByApp)...
// SurfaceTexture 继承自 ConsumerBase，最终由代理通知 FrameAvailable 
// frameworks\native\libs\gui\ConsumerBase.cpp
ConsumerBase::ConsumerBase(const sp<IGraphicBufferConsumer>& bufferQueue, bool controlledByApp) :
        mAbandoned(false),
        mConsumer(bufferQueue),
        mPrevFinalReleaseFence(Fence::NO_FENCE) {

    mName = String8::format("unnamed-%d-%d", getpid(), createProcessUniqueId());

    // 我们不能在构造函数中创建一个不会在构造函数结束后保持引用的sp<...>(this)，
    // 因为这会导致this的引用计数在构造函数结束时变为0。
    // 由于我们所需要的只是一个wp<...>，所以我们创建了它。
    wp<ConsumerListener> listener = static_cast<ConsumerListener*>(this);
    sp<IConsumerListener> proxy = new BufferQueue::ProxyConsumerListener(listener);

    // 创建一个消费者代理，并连接到代理 
    status_t err = mConsumer->consumerConnect(proxy, controlledByApp);
    if (err != NO_ERROR) {
        CB_LOGE("ConsumerBase: error connecting to BufferQueue: %s (%d)",
                strerror(-err), err);
    } else {
        mConsumer->setConsumerName(mName);
    }
}
```
本小节主要分析了 SurfaceTexture 初始化 和 onFrameAvailable 回调源码。
下小节主要分析 消费者 的 onFrameAvailable 在哪被调用的。

## Surface 绘制流程源码分析  
先看一张图，后面对图进行解释  
![](https://pic.chenjim.com/202401251805908.png-blog)  
原图来自 [Android绘制机制以及Surface家族源码全解析][1397749]  
- 首先我们从 lockCanvas 这个入口开始调用链是：lockCanvas-->nativeLockCanvas-->Surface::lock-->Surface::dequeueBuffer，这里最终会使用我们在 Surface 创建的时候得到的 BufferQueueProducer(IGraphicBufferProducer) 来想 SF (SurfaceFlinger)  请求一块空白的图像内存。得到了图像内存之后，将内存传入 new SkBitmap.cpp 中创建对应对象，一共后面使用 Skia 库绘制图像。这里的 SkBitmap.cpp 会被交给 SkCanvas.cpp 而 SkCanvas.cpp 对象就是 Canvas.java 在 c++ 层的操作对象。
- 上面我们通过 lockCanvas 获取到了一个 Canvas 对象。当我们调用 Canvas 的各种 api 的时候其实最终会代用到 c++ 层的 Skia 库，通过 cpu 对图像内存进行绘制。
- 当我们绘制完之后就可以调用 unlockCanvasAndPost 来通知 SF 合成图像，调用链是：unlockCanvasAndPost-->nativeUnlockCanvasAndPost-->Surface::queueBuffer，与 lockCanvas 中相反这里是最终是通过 BufferQueueProducer::queueBuffer 将图像内存放回到队列中，除此之外这里还调用 `IConsumerListener::onFrameAvailable` 来通知 SF 进程来刷新图像，调用链是：onFrameAvailable-->SF.signalLayerUpdate-->MQ.invalidate-->MQ.Handler.dispatchInvalidate-->MQ.Handler.handleMessage-->onMessageReceived：因为 SF 进程采用的也是事件驱动模型，所以这里和 ui thread 类似也是通过 looper + 事件 的形式触发 SurfaceFinger 对图像的刷新的。注意：这里的 IConsumerListener 是在 createNormalLayer 的时候创建的 Layer.cpp。  

这里没有贴出详细源码，可以依据名称，查看对应源码。  
到这里我们已经清晰了上一下小结中 onFrameAvailable 是从哪里回调来的。

## createBufferQueue 源码分析 
createBufferQueue 的实现：
```cpp
// frameworks/native/libs/gui/BufferQueue.cpp
// 创建 BufferQueue 进而初始化 producer 和 consumer
// camera如有帧数据来时，会调用dequeBuffer取一个buffer塞数据，塞好后再queueBuffer将其丢回到BufferQueue，然后通知consume
void BufferQueue::createBufferQueue(sp<IGraphicBufferProducer>* outProducer,
        sp<IGraphicBufferConsumer>* outConsumer,
        bool consumerIsSurfaceFlinger) {
    sp<BufferQueueCore> core(new BufferQueueCore());
    sp<IGraphicBufferProducer> producer(new BufferQueueProducer(core, consumerIsSurfaceFlinger));
    sp<IGraphicBufferConsumer> consumer(new BufferQueueConsumer(core));
    *outProducer = producer;
    *outConsumer = consumer;
}
```
![](https://pic.chenjim.com/202401251740430.png-blog)  
- 最左边表示原始图像流的生成方式：Video(本地/网络视频流)、Camera(摄像头拍摄的视频流)、Software/Hardware Render(使用 Skia/GL 绘制的图像流)。
- 最左边的原始图像流可以被交给 Surface，而 Surface 是 BQ 的生产者，GLConsumer(java层的体现就是 ST) 是 BQ 的消费者。
- GLConsumer 拿到了 Surface 的原始图像流，可以通过 GL 转化为 texture。最终以合理的方式消耗掉。
- 消耗 GLConsumer 中的 texture 的方式就是使用 SurfaceView、TextureView 等方式或者显示在屏幕上或者用于其他地方。

## SurfaceTexture 之 updateTexImage 源码分析 
通过前面初始化源码分析，我们知道会调用到 SurfaceTexture.cpp 的 updateTexImage  
```cpp
// frameworks\native\libs\nativedisplay\surfacetexture\SurfaceTexture.cpp
status_t SurfaceTexture::updateTexImage() {
        Mutex::Autolock lock(mMutex);
        return mEGLConsumer.updateTexImage(*this);
}
//frameworks/native/libs/nativedisplay/surfacetexture/EGLConsumer.cpp
status_t EGLConsumer::updateTexImage(SurfaceTexture& st) {
    // 检查EGL状态
    status_t err = checkAndUpdateEglStateLocked(st);

    BufferItem item;
    // 调用 status_t SurfaceTexture::acquireBufferLocked(*) 获取 BufferQueue 头部最新的图像内存
    err = st.acquireBufferLocked(&item, 0);

    // 释放之前的buffer
    err = updateAndReleaseLocked(item, nullptr, st);

    // 绑定新的 Buffer 到 GL texture，然后等待下一帧
    return bindTextureImageLocked(st);
}
status_t SurfaceTexture::acquireBufferLocked(BufferItem* item, nsecs_t presentWhen, uint64_t maxFrameNumber) {
    status_t err = ConsumerBase::acquireBufferLocked(item, presentWhen, maxFrameNumber);
    return NO_ERROR;
}
status_t ConsumerBase::acquireBufferLocked(BufferItem *item,
        nsecs_t presentWhen, uint64_t maxFrameNumber) {
    ...
    status_t err = mConsumer->acquireBuffer(item, presentWhen, maxFrameNumber);
    ...

    return OK;
}
status_t BufferQueueConsumer::acquireBuffer(BufferItem* outBuffer,
        nsecs_t expectedPresent, uint64_t maxFrameNumber) {.....}
```

---

## 结尾
到此我们分析了 SurfaceTexture 整个显示流程，可以用下图概括：  
![](https://pic.chenjim.com/202401251731753.png-blog)  
原图及本文部分内容来自 [Android绘制机制以及Surface家族源码全解析][1397749] , 感谢 ！！   
希望本文对你有帮助，欢迎留言讨论。


[1397749]: https://cloud.tencent.com/developer/article/1397749


