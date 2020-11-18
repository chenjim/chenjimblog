**原文** [一篇文章看明白Android图形系统Surface与SurfaceFlinger之间的关系](https://blog.csdn.net/freekiteyu/article/details/79483406)  

## Android - SurfaceFlinger 图形系统

### 相关系列

  * [一篇文章看明白 Android 系统启动时都干了什么](https://blog.csdn.net/freekiteyu/article/details/79175010)
  * [一篇文章了解相见恨晚的 Android Binder 进程间通讯机制](https://blog.csdn.net/freekiteyu/article/details/70082302)
  * [一篇文章看明白 Android 从点击应用图标到界面显示的过程](https://blog.csdn.net/freekiteyu/article/details/79318031)
  * [一篇文章看明白 Activity 与 Window 与 View 之间的关系](https://blog.csdn.net/freekiteyu/article/details/79408969)
  * [一篇文章看明白 Android 图形系统 Surface 与 SurfaceFlinger 之间的关系](https://blog.csdn.net/freekiteyu/article/details/79483406)
  * [一篇文章看明白 Android Service 启动过程](https://blog.csdn.net/freekiteyu/article/details/79785720)
  * [一篇文章看明白 Android PackageManagerService 工作流程](https://blog.csdn.net/freekiteyu/article/details/82774947)
  * [一篇文章看明白 Android v1 & v2 签名机制](https://blog.csdn.net/freekiteyu/article/details/84849651)

### 概述

  * [Android 系统启动过程](https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode/blob/master/android/Android-%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.md)
  * [Activity 创建过程](https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode/blob/master/android/Android-Activity%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.md)
  * [Activity 与 Window 与 View 之间的关系](https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode/blob/master/android/Android-Activity%E4%B8%8EWindow%E4%B8%8EView%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB.md)

![这里写图片描述](https://img-
blog.csdn.net/2018030813463528?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

通过前面的知识我们知道了，Android 系统从按下开机键到桌面，从桌面点击 App 图标到 Activity 显示的过程。但是 Activity
是怎么显示在屏幕上的呢？下面我们就来讨论下这一过程。

### SurfaceFlinger 启动过程

SurfaceFlinger 启动过程：

![这里写图片描述](https://img-
blog.csdn.net/2018030813470514?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

SurfaceFlinger 进程是由 init 进程创建的，运行在独立的 SurfaceFlinger 进程中。init 进程读取 init.rc
文件启动 SurfaceFlinger。

    
    
    service surfaceflinger /system/bin/surfaceflinger
        class core
        user system
        group graphics drmrpc
        onrestart restart zygote
        writepid /dev/cpuset/system-background/tasks
    

SurfaceFlinger 的创建会执行 main() 方法：  
main_surfaceflinger.cpp

    
    
    int main(int, char**) {
        ProcessState::self()->setThreadPoolMaxThreadCount(4);
    
        sp ps(ProcessState::self());
        ps->startThreadPool();
    
        //实例化 surfaceflinger
        sp flinger =  new SurfaceFlinger();
    
        setpriority(PRIO_PROCESS, 0, PRIORITY_URGENT_DISPLAY);
        set_sched_policy(0, SP_FOREGROUND);
    
        //初始化
        flinger->init();
    
        //发布 surface flinger，注册到 ServiceManager
        sp sm(defaultServiceManager());
        sm->addService(String16(SurfaceFlinger::getServiceName()), flinger, false);
    
        // 运行在当前线程
        flinger->run();
    
        return 0;
    }
    

SurfaceFlinger 的实例化会执行到：onFirstRef()

    
    
    void SurfaceFlinger::onFirstRef() {
        mEventQueue.init(this);
    }
    

onFirstRef() 中会创建 Handler 并初始化。  
MessageQueue.cpp：

    
    
    void MessageQueue::init(const sp& flinger) {
        mFlinger = flinger;
        mLooper = new Looper(true);
        mHandler = new Handler(*this);
    }
    

然后会执行到 SurfaceFlinger::init()：

    
    
    void SurfaceFlinger::init() {
        Mutex::Autolock _l(mStateLock);
    
        //初始化 EGL，作为默认的显示
        mEGLDisplay = eglGetDisplay(EGL_DEFAULT_DISPLAY);
        eglInitialize(mEGLDisplay, NULL, NULL);
    
        // 初始化硬件 composer 对象
        mHwc = new HWComposer(this, *static_cast(this));
    
        //获取 RenderEngine 引擎
        mRenderEngine = RenderEngine::create(mEGLDisplay, mHwc->getVisualID());
    
        //检索创建的 EGL 上下文
        mEGLContext = mRenderEngine->getEGLContext();
    
        //初始化非虚拟显示屏
        for (size_t i=0 ; iisConnected(i) || type==DisplayDevice::DISPLAY_PRIMARY) {
                bool isSecure = true;
                createBuiltinDisplayLocked(type);
                wp token = mBuiltinDisplays[i];
    
                sp producer;
                sp consumer;
                //创建 BufferQueue 的生产者和消费者
                BufferQueue::createBufferQueue(&producer;, &consumer;,
                        new GraphicBufferAlloc());
    
                sp fbs = new FramebufferSurface(*mHwc, i, consumer);
                int32_t hwcId = allocateHwcDisplayId(type);
                //创建显示设备
                sp hw = new DisplayDevice(this,
                        type, hwcId, mHwc->getFormat(hwcId), isSecure, token,
                        fbs, producer,
                        mRenderEngine->getEGLConfig());
                if (i > DisplayDevice::DISPLAY_PRIMARY) {
                    hw->setPowerMode(HWC_POWER_MODE_NORMAL);
                }
                mDisplays.add(token, hw);
            }
        }
    
        getDefaultDisplayDevice()->makeCurrent(mEGLDisplay, mEGLContext);
    
        //当应用和 sf 的 vsync 偏移量一致时，则只创建一个 EventThread 线程
        if (vsyncPhaseOffsetNs != sfVsyncPhaseOffsetNs) {
            sp vsyncSrc = new DispSyncSource(&mPrimaryDispSync;,
                    vsyncPhaseOffsetNs, true, "app");
            mEventThread = new EventThread(vsyncSrc);
            sp sfVsyncSrc = new DispSyncSource(&mPrimaryDispSync;,
                    sfVsyncPhaseOffsetNs, true, "sf");
            mSFEventThread = new EventThread(sfVsyncSrc);
            mEventQueue.setEventThread(mSFEventThread);
        } else {
            //创建 DispSyncSource 对象
            sp vsyncSrc = new DispSyncSource(&mPrimaryDispSync;,
                    vsyncPhaseOffsetNs, true, "sf-app");
            //创建线程 EventThread
            mEventThread = new EventThread(vsyncSrc);
            //设置 EventThread
            mEventQueue.setEventThread(mEventThread);
        }
    
        //创建 EventControl
        mEventControlThread = new EventControlThread(this);
        mEventControlThread->run("EventControl", PRIORITY_URGENT_DISPLAY);
    
        //当不存在 HWComposer 时，则设置软件 vsync
        if (mHwc->initCheck() != NO_ERROR) {
            mPrimaryDispSync.setPeriod(16666667);
        }
    
        //初始化绘图状态
        mDrawingState = mCurrentState;
    
        //初始化显示设备
        initializeDisplays();
    
        //启动开机动画
        startBootAnim();
    }
    

该方法主要功能是：

  1. 初始化 EGL
  2. 创建 HWComposer
  3. 初始化非虚拟显示屏
  4. 启动 EventThread 线程
  5. 启动开机动画

创建 HWComposer：

    
    
    HWComposer::HWComposer(const sp& flinger, EventHandler& handler):mFlinger(flinger), mFbDev(0), mHwc(0), mNumDisplays(1), mCBContext(new cb_context), mEventHandler(handler), mDebugForceFakeVSync(false) {
        ...
        bool needVSyncThread = true;
        int fberr = loadFbHalModule(); //加载 framebuffer 的 HAL 层模块
        loadHwcModule(); //加载 HWComposer 模块
    
        //标记已分配的 display ID
        for (size_t i=0 ; iregisterProcs) {
                mCBContext->hwc = this;
                mCBContext->procs.invalidate = &hook;_invalidate;
                //VSYNC 信号的回调方法
                mCBContext->procs.vsync = &hook;_vsync;
                if (hwcHasApiVersion(mHwc, HWC_DEVICE_API_VERSION_1_1))
                    mCBContext->procs.hotplug = &hook;_hotplug;
                else
                    mCBContext->procs.hotplug = NULL;
                memset(mCBContext->procs.zero, 0, sizeof(mCBContext->procs.zero));
                //注册回调函数
                mHwc->registerProcs(mHwc, &mCBContext-;>procs);
            }
    
            //进入此处，说明已成功打开硬件 composer 设备，则不再需要 vsync 线程
            needVSyncThread = false;
            eventControl(HWC_DISPLAY_PRIMARY, HWC_EVENT_VSYNC, 0);
            ...
        }
        ...
        if (needVSyncThread) {
            //不支持硬件的 VSYNC，则会创建线程来模拟定时 VSYNC 信号
            mVSyncThread = new VSyncThread(*this);
        }
    }
    

HWComposer 代表着硬件显示设备，注册了 VSYNC 信号的回调。VSYNC 信号本身是由显示驱动产生的，在不支持硬件的
VSYNC，则会创建“VSyncThread”线程来模拟定时 VSYNC 信号。

当硬件产生VSYNC信号时，则会发送消息，handler 收到消息进行处理。当 SurfaceFlinger 进程收到 VSync
信号后经层层调用，最终调用到该对象的 handleMessageRefresh() 方法。

SurfaceFlinger.cpp：

    
    
    void SurfaceFlinger::handleMessageRefresh() {
        ATRACE_CALL();
        preComposition();//处理显示设备与 layers 的改变，更新光标
        rebuildLayerStacks();//重建所有可见 Layer 列表，根据Z轴排序
        setUpHWComposer();//更新 HWComposer 图层
        doDebugFlashRegions(); 
        doComposition();//生成 OpenGL 纹理图像
        postComposition();//将图像传递到物理屏幕
    }
    

### Surface 创建过程

Surface 创建过程：  
![这里写图片描述](https://img-
blog.csdn.net/20180308134731851?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

Surface 创建的过程就是 Activity 显示的过程，在 ActivityThread.handleResumeActivity() 中调用了
Activity.makeVisible()，我们接着看下 Activity 是怎么显示出来的。

Activity.makeVisible：

    
    
    void makeVisible() {
        if (!mWindowAdded) {
            ViewManager wm = getWindowManager();//此处 getWindowManager 获取的是 WindowManagerImpl 对象
            wm.addView(mDecor, getWindow().getAttributes());
            mWindowAdded = true;
        }
        mDecor.setVisibility(View.VISIBLE);
    }
    

WindowManagerImpl.java：

    
    
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mDisplay, mParentWindow);
    }
    

WindowManagerGlobal.java：

    
    
    public void addView(View view, ViewGroup.LayoutParams params, Display display, Window parentWindow) {
        ...
        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
        //创建 ViewRootImpl
        ViewRootImpl root = new ViewRootImpl(view.getContext(), display);
        view.setLayoutParams(wparams);
        mViews.add(view);
        mRoots.add(root);
        mParams.add(wparams);
        //设置 View
        root.setView(view, wparams, panelParentView);
        ...
    }
    

创建 ViewRootImpl：

    
    
    public final class ViewRootImpl implements ViewParent,
            View.AttachInfo.Callbacks, ThreadedRenderer.DrawCallbacks {
        ...
        final Surface mSurface = new Surface(); //创建 Surface，此时 Surface 创建完什么都没有，详见下面分析
        ...
        public ViewRootImpl(Context context, Display display) {
            mContext = context;
            //获取 IWindowSession 的代理类
            mWindowSession = WindowManagerGlobal.getWindowSession();
            mDisplay = display;
            mThread = Thread.currentThread(); //主线程
            mWindow = new W(this);
            mChoreographer = Choreographer.getInstance();
            ...
        }
    }
    

WindowManagerGlobal.java：

    
    
    public static IWindowSession getWindowSession() {
        synchronized (WindowManagerGlobal.class) {
            if (sWindowSession == null) {
                try {
                    //获取 IMS 的代理类
                    InputMethodManager imm = InputMethodManager.getInstance();
                    //获取 WMS 的代理类
                    IWindowManager windowManager = getWindowManagerService();
                    //经过 Binder 调用，最终调用 WMS
                    sWindowSession = windowManager.openSession(
                            new IWindowSessionCallback.Stub() {...},
                            imm.getClient(), imm.getInputContext());
                } catch (RemoteException e) {
                    ...
                }
            }
            return sWindowSession
        }
    }
    

WindowManagerService.openSession：

    
    
    public IWindowSession openSession(IWindowSessionCallback callback, IInputMethodClient client, IInputContext inputContext) {
        //创建 Session 对象
        Session session = new Session(this, callback, client, inputContext);
        return session;
    }
    

再次经过 Binder 将数据写回 app 进程，则获取的便是 Session 的代理对象 IWindowSession。

创建完 ViewRootImpl 对象后，接下来调用该对象的 setView() 方法。  
ViewRootImpl：

    
    
    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
      synchronized (this) {
      
        requestLayout(); //详见下面分析
        ...
        //通过 Binder调用，进入 system 进程的 Session
        res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
              getHostVisibility(), mDisplay.getDisplayId(),
              mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
              mAttachInfo.mOutsets, mInputChannel);
        ...
      }
    }
    
    
    
    final class Session extends IWindowSession.Stub implements IBinder.DeathRecipient {
    
        public int addToDisplay(IWindow window, int seq, WindowManager.LayoutParams attrs, int viewVisibility, int displayId, Rect outContentInsets, Rect outStableInsets, Rect outOutsets, InputChannel outInputChannel) {
            //调用 WMS.addWindow
            return mService.addWindow(this, window, seq, attrs, viewVisibility, displayId,
                    outContentInsets, outStableInsets, outOutsets, outInputChannel);
        }
    }
    

WindowManagerService.java：

    
    
    public int addWindow(Session session, IWindow client, int seq, WindowManager.LayoutParams attrs, int viewVisibility, int displayId, Rect outContentInsets, Rect outStableInsets, Rect outOutsets, InputChannel outInputChannel) {
        ...
        WindowToken token = mTokenMap.get(attrs.token);
        //创建 WindowState
        WindowState win = new WindowState(this, session, client, token,
                    attachedWindow, appOp[0], seq, attrs, viewVisibility, displayContent);
        ...
        //调整 WindowManager 的 LayoutParams 参数
        mPolicy.adjustWindowParamsLw(win.mAttrs);
        res = mPolicy.prepareAddWindowLw(win, attrs);
        addWindowToListInOrderLocked(win, true);
        // 设置 input
        mInputManager.registerInputChannel(win.mInputChannel, win.mInputWindowHandle);
        //详见下面分析
        win.attach();
        mWindowMap.put(client.asBinder(), win);
        
        if (win.canReceiveKeys()) {
            //当该窗口能接收按键事件，则更新聚焦窗口
            focusChanged = updateFocusedWindowLocked(UPDATE_FOCUS_WILL_ASSIGN_LAYERS,
                    false /*updateInputWindows*/);
        }
        assignLayersLocked(displayContent.getWindowList());
        ...
    }
    
    //WindowState.java
    void attach() {
        mSession.windowAddedLocked();
    }
    

创建 SurfaceSession 对象，并将当前 Session 添加到 WMS.mSessions 成员变量。  
Session.java：

    
    
    void windowAddedLocked() {
        if (mSurfaceSession == null) {
            mSurfaceSession = new SurfaceSession();
            mService.mSessions.add(this);
            if (mLastReportedAnimatorScale != mService.getCurrentAnimatorScale()) {
                mService.dispatchNewAnimatorScaleLocked(this);
            }
        }
        mNumWindow++;
    }
    

SurfaceSession 的创建会调用 JNI，在 JNI 调用 nativeCreate()。  
android_view_SurfaceSession.cpp：

    
    
    static jlong nativeCreate(JNIEnv* env, jclass clazz) {
        SurfaceComposerClient* client = new SurfaceComposerClient();
        client->incStrong((void*)nativeCreate);
        return reinterpret_cast(client);
    }
    

创建 SurfaceComposerClient 对象， 作为跟 SurfaceFlinger 通信的代理对象。

    
    
    SurfaceComposerClient::SurfaceComposerClient() {
        //getComposerService() 将返回 SF 的 Binder 代理端的 BpSurfaceFlinger 对象
        sp sm(getComposerService());
        
        //先调用 SF 的 createConnection()，再调用_init
        _init(sm, sm->createConnection());
        if(mClient != 0) {
           Mutex::Autolock _l(gLock);
           
           //gActiveConnections 是全局变量，把刚才创建的 client 保存到这个 map 中去
           gActiveConnections.add(mClient->asBinder(), this);
        }
    }
    

SurfaceFlinger.cpp：

    
    
    spSurfaceFlinger::createConnection() {
        Mutex::Autolock _l(mStateLock);
        uint32_t token = mTokens.acquire();
    
        //先创建一个Client
        sp client = new Client(token, this);
    
        //把这个Client对象保存到mClientsMap中，token是它的标识。
        status_t err = mClientsMap.add(token, client);
    
        /*
        创建一个用于 Binder 通信的 BClient，BClient 派生于 ISurfaceFlingerClient，
        它的作用是接受客户端的请求，然后把处理提交给 SF，注意，并不是提交给 Client。
        Client 会创建一块共享内存，该内存由 getControlBlockMemory 函数返回。
        */
        sp bclient = new BClient(this, token,client->getControlBlockMemory());
        return bclient;
    }
    
    
    Client::Client(ClientID clientID, constsp& flinger):ctrlblk(0), cid(clientID), mPid(0), mBitmap(0), mFlinger(flinger) {
    const int pgsize = getpagesize();
        //下面这个操作会使 cblksize 为页的大小，目前是4096字节
        constint cblksize = ((sizeof(SharedClient)+(pgsize-1))&~(pgsize-1));
        mCblkHeap = new MemoryHeapBase(cblksize, 0, "SurfaceFlinger Clientcontrol-block");
    
        ctrlblk = static_cast(mCblkHeap->getBase());
        if(ctrlblk) {
           new(ctrlblk) SharedClient;//原来 Surface 的 CB 对象就是在共享内存中创建的这个 SharedClient 对象
        }
    }
    

SharedClient：

    
    
    class SharedClient {
    
    public:
       SharedClient();
       ~SharedClient();
       status_t validate(size_t token) const;
       uint32_t getIdentity(size_t token) const;
    
    private:
        Mutexlock;
        Condition cv; //支持跨进程的同步对象
    
        //NUM_LAYERS_MAX 为 31，SharedBufferStack 是什么？
        SharedBufferStack surfaces[ NUM_LAYERS_MAX ];
    
    };
    
    //SharedClient的构造函数，没什么新意，不如Audio的CB对象复杂
    SharedClient::SharedClient():lock(Mutex::SHARED), cv(Condition::SHARED) {
    }
    

一个 Client 最多支持 31 个显示层。每一个显示层的生产/消费步调都由会对应的 SharedBufferStack
来控制。而它内部就用了几个成员变量来控制读写位置。

SharedBufferStack.h：

    
    
    class  SharedBufferStack{
         ......
        //Buffer 是按块使用的，每个 Buffer 都有自己的编号，其实就是数组中的索引号。
        volatile int32_t head;     //FrontBuffer 的编号
        volatile int32_t available; //空闲 Buffer 的个数
        volatile int32_t queued;  //脏 Buffer 的个数，脏 Buffer 表示有新数据的 Buffer
        volatile int32_t inUse; //SF 当前正在使用的 Buffer 的编号   
        volatilestatus_t status; //状态码
         ......
      }
    

SF 的一个 Client 分配一个跨进程共享的 SharedClient 对象。这个对象有31个 SharedBufferStack 元素，每一个
SharedBufferStack 对应于一个显示层。

一个显示层将创建两个 Buffer，后续的 PageFlipping 就是基于这两个 Buffer 展开的。

接着看 SurfaceComposerClient 中这个_init函数：

    
    
    void SurfaceComposerClient::_init(
           const sp& sm, constsp& conn) {
        mPrebuiltLayerState = 0;
        mTransactionOpen = 0;
        mStatus = NO_ERROR;
        mControl = 0;
    
        mClient = conn;// mClient 就是 BClient 的客户端
        mControlMemory =mClient->getControlBlock();
        mSignalServer = sm;// mSignalServer 就是 BpSurfaceFlinger
        //mControl 就是那个创建于共享内存之中的 SharedClient
        mControl = static_cast(mControlMemory->getBase());
    }
    

创建完 ViewRootImpl 对象后，接下来调用该对象的 setView() 方法。在 setView() 中调用了 requestLayout()
方法我们来看下这个方法：

    
    
    public void requestLayout() {
       checkThread();
       mLayoutRequested = true;
       scheduleTraversals();
    }
    
    public void scheduleTraversals() {
        if(!mTraversalScheduled) {
           mTraversalScheduled = true;
           sendEmptyMessage(DO_TRAVERSAL); //发送 DO_TRAVERSAL 消息
        }
    }
    
    public void handleMessage(Message msg) {
       switch (msg.what) {
        ......
        case DO_TRAVERSAL:
            ......
            performTraversals();//调用 performTraversals()
            ......
            break;
        ......
        }
    }
    
    private void performTraversals() {
        finalView host = mView;//还记得这mView吗？它就是 DecorView
        booleaninitialized = false;
        booleancontentInsetsChanged = false;
        booleanvisibleInsetsChanged;
        
        try {
            relayoutResult= // 1. 关键函数relayoutWindow
            relayoutWindow(params, viewVisibility,insetsPending);
        }
        ......
        draw(fullRedrawNeeded);// 2. 开始绘制
        ......
    }
    
    private int relayoutWindow(WindowManager.LayoutParams params, int viewVisibility, boolean insetsPending)throws RemoteException {
           //原来是调用 IWindowSession 的 relayout()，暂且记住这个调用
           int relayoutResult = sWindowSession.relayout(mWindow, params, (int) (mView.mMeasuredWidth * appScale + 0.5f),  (int) (mView.mMeasuredHeight * appScale + 0.5f), viewVisibility, insetsPending, mWinFrame, mPendingContentInsets, mPendingVisibleInsets, mPendingConfiguration, mSurface); //mSurface 做为参数传进去了。
           }
       ......
    }
    
    private void draw(boolean fullRedrawNeeded) {
        Surface surface = mSurface;//mSurface 是 ViewRoot 的成员变量
        ......
        Canvascanvas;
    
        try {
           int left = dirty.left;
           int top = dirty.top;
           int right = dirty.right;
           int bottom = dirty.bottom;
    
           //从 mSurface 中 lock 一块 Canvas
           canvas = surface.lockCanvas(dirty);
           ......
           mView.draw(canvas);//调用 DecorView 的 draw 函数，canvas 就是画布
           ......
           //unlock 画布，屏幕上马上就能看到 View 的样子了
           surface.unlockCanvasAndPost(canvas);
        }
        ......
    }
    

在 ViewRoot 构造时，会创建一个 Surface，它使用无参构造函数，代码如下所示：

    
    
    final Surface mSurface = new Surface();
    

此时创建完的 Surface 是空的，什么都没有。接着继续分析 relayoutWindow()，在 relayoutWindow() 中会调用
IWindowSession 的 relayout()，这是一个跨进程方法会调用到 WMS 中的 Session.relayout()，最后调用到
WindowManagerService.relayoutWindow()。

    
    
    public int relayoutWindow(Session session,IWindow client,
               WindowManager.LayoutParams attrs, int requestedWidth,
               int requestedHeight, int viewVisibility, boolean insetsPending,
               Rect outFrame, Rect outContentInsets, Rect outVisibleInsets,
                Configuration outConfig, SurfaceoutSurface){
            .....
    
        try {
             //win 就是 WinState，这里将创建一个本地的 Surface 对象
            Surfacesurface = win.createSurfaceLocked();
            if(surface != null) {
                //先创建一个本地 surface，然后在 outSurface 的对象上调用 copyFrom
                //将本地 Surface 的信息拷贝到 outSurface 中，为什么要这么麻烦呢？
                outSurface.copyFrom(surface);
            ......
    }
    

WindowManagerService.java::WindowState：

    
    
    Surface createSurfaceLocked() {
        ......
        try {
            //mSurfaceSession 就是在 Session 上创建的 SurfaceSession 对象
            //这里，以它为参数，构造一个新的 Surface 对象
            mSurface = new Surface(mSession.mSurfaceSession, mSession.mPid, mAttrs.getTitle().toString(), 0, w, h, mAttrs.format, flags);
        }
        Surface.openTransaction();//打开一个事务处理
        ......
        Surface.closeTransaction();//关闭一个事务处理
        ......
    }
    

构造 Surface 对象：

    
    
     public Surface(SurfaceSession s,//传入一个SurfaceSession对象
        int pid, String name, int display, int w, int h, int format, int flags) throws OutOfResourcesException {
            ......
            mCanvas = new CompatibleCanvas();
            //又一个 native 函数
            init(s,pid,name,display,w,h,format,flags);
            mName = name;
        }
    
    
    
    static void Surface_init(JNIEnv*env, jobject clazz, jobject session, jint pid, jstring jname, jint dpy, jint w, jint h, jint format, jintflags) {
    
        //从 SurfaceSession 对象中取出之前创建的那个 SurfaceComposerClient 对象
        SurfaceComposerClient* client = (SurfaceComposerClient*)env->GetIntField(session, sso.client);
        sp surface;//注意它的类型是 SurfaceControl
        if (jname == NULL) {
            //调用 SurfaceComposerClient 的 createSurface 函数，返回的 surface 是一个 SurfaceControl 类型
            surface = client->createSurface(pid, dpy, w, h, format, flags);
        } else{
            ......
        }
    
       //把这个 surfaceControl 对象设置到 Java 层的 Surface 对象中
       setSurfaceControl(env, clazz, surface);
    }
    

在 createSurface 内部会使用 Binder 通信将请求发给 SurfaceFlinger：

    
    
    spSurfaceFlinger::createSurface(ClientID clientId, int pid, const String8& name, ISurfaceFlingerClient::surface_data_t* params, DisplayID d, uint32_t w, uint32_t h, PixelFormat format, uint32_t flags) {
        sp layer;//LayerBaseClient 是 Layer 家族的基类
        //这里又冒出一个 LayerBaseClient 的内部类，它也叫Surface
        sp surfaceHandle;
        Mutex::Autolock _l(mStateLock);
    
        //根据 clientId 找到 createConnection 时加入的那个 Client 对象
        sp client = mClientsMap.valueFor(clientId);
        ......
        //注意这个 id，它的值表示 Client 创建的是第几个显示层
        //同时也表示将使用 SharedBufferStatck 数组的第 id 个元素
        int32_t id = client->generateId(pid);
        
        //一个 Client 不能创建多于 NUM_LAYERS_MAX 个的Layer
        if(uint32_t(id) >= NUM_LAYERS_MAX) {
           return surfaceHandle;
        }
    
        //根据 flags 参数来创建不同类型的显示层
        switch(flags & eFXSurfaceMask) {
            case eFXSurfaceNormal:
               if (UNLIKELY(flags & ePushBuffers)) {
                 //创建 PushBuffer 类型的显示层
                layer = createPushBuffersSurfaceLocked(client, d, id, w, h, flags);
                } else {
                   //创建 Normal 类型的显示层
                   layer = createNormalSurfaceLocked(client, d, id, w, h, flags, format);
               }
               break;
            case eFXSurfaceBlur:
                //创建 Blur 类型的显示层
               layer = createBlurSurfaceLocked(client, d, id, w, h, flags);
               break;
            case eFXSurfaceDim:
                //创建 Dim 类型的显示层
               layer = createDimSurfaceLocked(client, d, id, w, h, flags);
               break;
        }
    
        if(layer != 0) {
            layer->setName(name);
            setTransactionFlags(eTransactionNeeded);
            //从显示层对象中取出一个 ISurface 对象赋值给 SurfaceHandle
            surfaceHandle = layer->getSurface();
            if(surfaceHandle != 0) {
               params->token = surfaceHandle->getToken();
               params->identity = surfaceHandle->getIdentity();
               params->width = w;
               params->height = h;
               params->format = format;
            }
        }
        return surfaceHandle;//ISurface 的 Bn 端就是这个对象
    }
    
    
    
    spSurfaceFlinger::createNormalSurfaceLocked(const sp& client, DisplayID display, int32_t id, uint32_t w, uint32_t h, uint32_t flags, PixelFormat& format) {
        switch(format) { //一些图像方面的参数设置，可以不去管它
        case PIXEL_FORMAT_TRANSPARENT:
        case PIXEL_FORMAT_TRANSLUCENT:
           format = PIXEL_FORMAT_RGBA_8888;
           break;
        case PIXEL_FORMAT_OPAQUE:
           format = PIXEL_FORMAT_RGB_565;
           break;
        }
    
        //创建一个 Layer 类型的对象
        sp layer = new Layer(this, display,client, id);
    
        //设置 Buffer
        status_t err = layer->setBuffers(w, h, format, flags);
        if (LIKELY(err == NO_ERROR)) {
            //初始化这个新 layer 的一些状态
            layer->initStates(w, h, flags);
            //下面这个函数把这个 layer 加入到 Z 轴集合中
            addLayer_l(layer);
        }
    ......
        return layer;
    }
    

createNormalSurfaceLocked 函数有三个关键点，它们是：

  * 构造一个Layer对象。
  * 调用Layer对象的setBuffers函数。
  * 调用SF的addLayer_l函数。

当跨进程的 createSurface() 执行完返回一个 ISurface 对象，接下来会创建 SurfaceControl 对象：

    
    
    SurfaceControl::SurfaceControl(
           const sp& client,
           const sp& surface,
           const ISurfaceFlingerClient::surface_data_t& data,
           uint32_t w, uint32_t h, PixelFormat format, uint32_t flags)
        //mClient 为 SurfaceComposerClient，而 mSurface 指向跨进程 createSurface() 调用返回的 ISurface 对象
        :mClient(client), mSurface(surface),
         mToken(data.token), mIdentity(data.identity),
         mWidth(data.width), mHeight(data.height), mFormat(data.format),
         mFlags(flags){
         ......
    }
    

SurfaceControl 类可以看作是一个 wrapper 类，它封装了一些函数，通过这些函数可以方便地调用 mClient 或 ISurface
提供的函数。

最后会执行 copyFrom() 返回给 App 客户端：

    
    
    static void Surface_copyFrom(JNIEnv* env,jobject clazz, jobject other) {
        //根据JNI函数的规则，clazz 是 copyFrom 的调用对象，而 other 是 copyFrom 的参数。
        //目标对象此时还没有设置 SurfaceControl，而源对象在前面已经创建了 SurfaceControl
        constsp& surface = getSurfaceControl(env, clazz);
        constsp& rhs = getSurfaceControl(env, other);
        if (!SurfaceControl::isSameSurface(surface, rhs)) {
            //把源 SurfaceControl 对象设置到目标 Surface 中
            setSurfaceControl(env, clazz, rhs);
        }
    }
    

copyFrom 期间一共有三个关键对象，它们分别是：

  * SurfaceComposerClient
  * SurfaceControl
  * Surface，这个 Surface 对象属于 Native 层，和 Java 层的 Surface 相对应

其中转移到 ViewRoot 成员变量 mSurface 中的，就是最后这个 Surface 对象了。

在 SurfaceFlinger 进程中，Client 的一个 Layer 将使用 SharedBufferStack 数组中的一个成员，并通过
SharedBufferServer 结构来控制这个成员，我们知道 SurfaceFlinger 是消费者，所以可由 SharedBufferServer
来控制数据的读取。

与之相对应，客户端的进程也会有一个对象来使用这个 SharedBufferStack，可它是通过另外一个叫 SharedBufferClient
的结构来控制的。客户端为 SurfaceFlinger 提供数据，所以可由 SharedBufferClient 控制数据的写入。

### Surface 显示过程

![这里写图片描述](https://img-
blog.csdn.net/20180308134806204?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如图所示，在 App 进程中创建 PhoneWindow 后会创建 ViewRoot。ViewRoot 的创建会创建一个 Surface，这个
Surface 其实是空的，通过与 WindowManagerService 通信 copyFrom() 一个 NativeSurface。在与
SurfaceFlinger 通信时，会创建 SharedClient 一段共享内存，里面存放的是 SharedBufferStack 对应
SurfaceFlinger 中的 SurfaceLayer 每个 Layer 其实是一个 FrameBuffer，每个 FrameBuffer 中有两个
GraphicBuffer 记作 FrontBuffer 和 BackBuffer。

在 SurfaceFlinger 中 SharedBufferServer 来管理 FrameBuffer。同时在 App 端 copyFrom() 出来
NativeSurface 时会创建一个 SharedBufferClient 与 SharedClient 这块共享内存关联。当客户端 addView()
或者需要更新 View 时，会通过 SharedBufferClient 写入数据到 ShareClient 中，SurfaceFlinger 中的
SharedBufferServer 接收到通知会将 FrameBuffer 中的数据传输到屏幕上。

HWComposer 是基于硬件来产生 VSync 信号的，来通知 SurfaceFlinger 重绘控制显示的帧率。

以上理解属于个人观点，能力有限，若有错误欢迎指出，欢迎交流学习，共同进步。

### 参考资料

  * [深入理解Surface系统](http://blog.csdn.net/innost/article/details/47208337)
  * [Android图形系统](http://gityuan.com/2017/02/05/graphic_arch/)
  * [SurfaceFlinger启动篇](http://gityuan.com/2017/02/05/graphic_arch/)
  * [SurfaceFlinger绘图篇](http://gityuan.com/2017/02/18/surface_flinger_2/)
  * 《深入理解 Android 内核设计思想》

### 其他系列

  * [Android 屏幕适配全攻略](https://blog.csdn.net/freekiteyu/article/details/70790711)
  * [Windows 环境下载 Android 源码](https://blog.csdn.net/freekiteyu/article/details/70939672)
  * [Android 性能优化-UI优化](https://blog.csdn.net/freekiteyu/article/details/77862670)
  * [Android 性能优化-内存优化](https://blog.csdn.net/freekiteyu/article/details/78060938)
  * [Java 虚拟机内存分配机制](https://blog.csdn.net/freekiteyu/article/details/77992277)
  * [Java 虚拟机垃圾回收机制](https://blog.csdn.net/freekiteyu/article/details/78029495)
  * [一篇文章看明白 TCP/IP，TCP，UDP，IP，Socket 之间的关系](https://blog.csdn.net/freekiteyu/article/details/72236734)
  * [一篇文章看明白 HTTP，HTTPS，SSL/TSL 之间的关系](https://blog.csdn.net/freekiteyu/article/details/76423436)

### Gradle 系列

  * [Gradle - 简介](https://blog.csdn.net/freekiteyu/article/details/80677361)
  * [Gradle - Groovy Language](https://blog.csdn.net/freekiteyu/article/details/80846149)
  * [Gradle - DSL](https://blog.csdn.net/freekiteyu/article/details/81066845)
  * [Gradle - Android Plugin DSL](https://blog.csdn.net/freekiteyu/article/details/81173683)
  * [Gradle - 插件开发](https://blog.csdn.net/freekiteyu/article/details/81353922)
  * [Gradle - 插件发布](https://blog.csdn.net/freekiteyu/article/details/81449721)

### 更多文章：

这是我博客长期更新的项目，欢迎大家 Star。  
<https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode>

### 我的公众号

欢迎你「扫一扫」下面的二维码，关注我的公众号，可以接受最新的文章推送，有丰厚的抽奖活动和福利等着你哦！?

![](https://raw.githubusercontent.com/jeanboydev/Android-
ReadTheFuckingSourceCode/master/resources/images/about_me/qrcode_android_besos_black_512.png)

如果你有什么疑问或者问题，可以 [点击这里](https://github.com/jeanboydev/Android-
ReadTheFuckingSourceCode/issues) 提交 issue，也可以发邮件给我
[jeanboy@foxmail.com](mailto:jeanboy@foxmail.com)。

同时欢迎你
[![Android技术进阶：386463747](https://camo.githubusercontent.com/615c9901677f501582b6057efc9396b3ed27dc29/687474703a2f2f7075622e69647171696d672e636f6d2f7770612f696d616765732f67726f75702e706e67)](http://shang.qq.com/wpa/qunwpa?idkey=0b505511df9ead28ec678df4eeb7a1a8f994ea8b75f2c10412b57e667d81b50d)
来一起交流学习，群里有很多大牛和学习资料，相信一定能帮助到你！

