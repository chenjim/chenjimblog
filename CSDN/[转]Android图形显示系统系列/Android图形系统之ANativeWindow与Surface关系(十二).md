**原文** [Android图形系统之ANativeWindow与Surface关系(十二)](https://unbroken.blog.csdn.net/article/details/131837369)  

**简介：** **CSDN博客专家，专注Android/Linux系统，分享多mic语音方案、音视频、编解码等技术，与大家一起成长！**

**优质专栏：**[**Audio工程师进阶系列**](https://blog.csdn.net/u010164190/category_9599435.html)【**原创干货持续更新中……**
】🚀

**人生格言：** **人生从来没有捷径，只有行动才是治疗恐惧和懒惰的唯一良药.**

更多原创,欢迎关注：Android系统攻城狮

![欢迎关注Android系统攻城狮](https://img-
blog.csdnimg.cn/20190106163945739.jpg#pic_center)

### 1.前言

> 本篇目的：理解Android显示系统之ANativeWindow与Surface关系

### 2.ANativeWindow与Surface类图

> frameworks/native/libs/gui/include/gui/Surface.h
    
    
    class Surface : public ANativeObjectBase<ANativeWindow, Surface, RefBase>
    {
      public:
      explicit Surface(const sp<IGraphicBufferProducer>& bufferProducer, bool controlledByApp = false,
                         const sp<IBinder>& surfaceControlHandle = nullptr);
      sp<IGraphicBufferProducer> getIGraphicBufferProducer() const;
      sp<IBinder> getSurfaceControlHandle() const { return mSurfaceControlHandle; }
      void setSidebandStream(const sp<NativeHandle>& stream);
      private:
        // can't be copied
        Surface& operator = (const Surface& rhs);
        Surface(const Surface& rhs);
    
        // ANativeWindow hooks
        static int hook_cancelBuffer(ANativeWindow* window,
                ANativeWindowBuffer* buffer, int fenceFd);
        static int hook_dequeueBuffer(ANativeWindow* window,
                ANativeWindowBuffer** buffer, int* fenceFd);
        static int hook_perform(ANativeWindow* window, int operation, ...);
        static int hook_query(const ANativeWindow* window, int what, int* value);
        static int hook_queueBuffer(ANativeWindow* window,
                ANativeWindowBuffer* buffer, int fenceFd);
        static int hook_setSwapInterval(ANativeWindow* window, int interval);
    
        static int cancelBufferInternal(ANativeWindow* window, ANativeWindowBuffer* buffer,
                                        int fenceFd);
        static int dequeueBufferInternal(ANativeWindow* window, ANativeWindowBuffer** buffer,
                                         int* fenceFd);
        static int performInternal(ANativeWindow* window, int operation, va_list args);
        static int queueBufferInternal(ANativeWindow* window, ANativeWindowBuffer* buffer, int fenceFd);
        static int queryInternal(const ANativeWindow* window, int what, int* value);
    
        static int hook_cancelBuffer_DEPRECATED(ANativeWindow* window,
                ANativeWindowBuffer* buffer);
        static int hook_dequeueBuffer_DEPRECATED(ANativeWindow* window,
                ANativeWindowBuffer** buffer);
        static int hook_lockBuffer_DEPRECATED(ANativeWindow* window,
                ANativeWindowBuffer* buffer);
        static int hook_queueBuffer_DEPRECATED(ANativeWindow* window,
                ANativeWindowBuffer* buffer);
                
    };
    

继承

ANativeObjectBase

Surface

+sp getIGraphicBufferProducer()

+sp getSurfaceControlHandle()

#int connect()

ANativeWindow

-int (*query)()

-int (*perform)()

-int (*queueBuffer)()

-int (*dequeueBuffer)()

### 3.ANativeWindow与Surface转换用法实例

#### <1>.用法一

    
    
    sp<SurfaceComposerClient> client = new SurfaceComposerClient;
    
    void render(onst sp<ANativeWindow> &nativeWindow) {
    (NativeWindow.get())->perform(NativeWindow.get(), NATIVE_WINDOW_SET_BUFFERS_GEOMETRY,bufWidth, bufHeight, halFormat);
    }
    
    
    sp<SurfaceControl> surfaceControl = client->createSurface(String8("testsurface"),lcd_width, lcd_height, PIXEL_FORMAT_RGBA_8888, 0);
    sp<Surface> surface = surfaceControl->getSurface();
    
    test(surface);							    
    
    

#### <2>.用法二

    
    
    ANativeWindow* window = nullptr;
    Surface* c = static_cast<Surface*>(window);
    
    

