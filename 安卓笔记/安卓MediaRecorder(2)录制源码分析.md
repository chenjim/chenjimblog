
[TOC]

>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/134634628>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>

## 前言
通过 [前文1](https://h89.cn/archives/77.html)，我们已经知道如何使用 MediaRecorder。
本文主要分析一下 Framework 中相关流程。   
下图是谷歌提供的 MediaRecorder 状态关系图   
![](https://pic.chenjim.com/202312142157524.gif-blog)

## JAVA new MediaRecorder() 源码分析 
```java
public class MediaRecorder implements AudioRouting,AudioRecordingMonitor,
        AudioRecordingMonitorClient,MicrophoneDirection {

    static {
        // 静态代码块，加载链接 liblibmedia_jni.so
        System.loadLibrary("media_jni");
        native_init();
    }

    // 已经废弃 
    public MediaRecorder() {
        // 传入 APP Context
        this(ActivityThread.currentApplication());
    }

    public MediaRecorder(@NonNull Context context) {
        // 要求 Context 不为空
        Objects.requireNonNull(context);

        // 创建EventHandler，主要用于JNI层回调时切换到当前App端线程中
        Looper looper;
        if ((looper = Looper.myLooper()) != null) {
            // 如果当前线程有Looper，就使用当前线程的Looper
            mEventHandler = new EventHandler(this, looper);
        } else if ((looper = Looper.getMainLooper()) != null) {
            // 使用主线程的 Looper，Jni回调信息会在主线程执行   
            mEventHandler = new EventHandler(this, looper);
        } else {
            mEventHandler = null;
        }

        // 录制音频的声道数，此处默认 1 (mono即单声道)，2 (stereo即双声道立体声)，可以通过 setAudioChannels 修改
        mChannelCount = 1;

        // 创建弱引用，并初始化
        try (ScopedParcelState attributionSourceState = context.getAttributionSource().asScopedParcelState()) {
            native_setup(new WeakReference<>(this), ActivityThread.currentPackageName(),
                    attributionSourceState.getParcel());
        }
    }
```
### android_media_MediaRecorder.cpp native_init() 
获取JAVA层 android.media.MediaRecorder 对象  
并将 JAVA 对象的属性 mNativeContext、mSurface、postEventFromNative 保存在 fields   
详细代码如下  
```cpp
static void
android_media_MediaRecorder_native_init(JNIEnv *env)
{
    jclass clazz;
   
    clazz = env->FindClass("android/media/MediaRecorder");
    if (clazz == NULL) {
        return;
    }

    fields.context = env->GetFieldID(clazz, "mNativeContext", "J");
    if (fields.context == NULL) {
        return;
    }

    fields.surface = env->GetFieldID(clazz, "mSurface", "Landroid/view/Surface;");
    if (fields.surface == NULL) {
        return;
    }

    jclass surface = env->FindClass("android/view/Surface");
    if (surface == NULL) {
        return;
    }

    fields.post_event = env->GetStaticMethodID(clazz, "postEventFromNative",
                                               "(Ljava/lang/Object;IIILjava/lang/Object;)V");
    if (fields.post_event == NULL) {
        return;
    }

    clazz = env->FindClass("java/util/ArrayList");
    if (clazz == NULL) {
        return;
    }
    gArrayListFields.add = env->GetMethodID(clazz, "add", "(Ljava/lang/Object;)Z");
    gArrayListFields.classId = static_cast<jclass>(env->NewGlobalRef(clazz));
}
```
### MediaRecorder.java postEventFromNative
这个是Jni消息回来的接口，最终会发到 MediaRecorder.EventHandler 的 handleMessage 中  
进而可以通过 MediaRecorder.OnInfoListener 、MediaRecorder.OnErrorListener、  
AudioRouting.OnRoutingChangedListener  回调到 APP   
对应源码如下  
```java
private static void postEventFromNative(Object mediarecorder_ref,
                                        int what, int arg1, int arg2, Object obj) {
    MediaRecorder mr = (MediaRecorder)((WeakReference)mediarecorder_ref).get();
    if (mr == null) {
        return;
    }

    if (mr.mEventHandler != null) {
        Message m = mr.mEventHandler.obtainMessage(what, arg1, arg2, obj);
        mr.mEventHandler.sendMessage(m);
    }
}

// EventHandler 如下  
public class MediaRecorder {
    ...
    private class EventHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
           switch(msg.what) {
            case MEDIA_RECORDER_EVENT_ERROR:
            case MEDIA_RECORDER_TRACK_EVENT_ERROR:
                if (mOnErrorListener != null)
                    mOnErrorListener.onError(mMediaRecorder, msg.arg1, msg.arg2);
                return;
            case MEDIA_RECORDER_EVENT_INFO:
            case MEDIA_RECORDER_TRACK_EVENT_INFO:
                if (mOnInfoListener != null)
                    mOnInfoListener.onInfo(mMediaRecorder, msg.arg1, msg.arg2);
                return;
            case MEDIA_RECORDER_AUDIO_ROUTING_CHANGED:
                // 耳机使能的消息  
                return;
        }
    }

}
```

### android_media_MediaRecorder.cpp native_setup() 
```cpp
static void
android_media_MediaRecorder_native_setup(JNIEnv *env, jobject thiz, jobject weak_this,
                                         jstring packageName, jobject jAttributionSource)
{
    // 构建 JNI 对象 MediaRecorder，attributionSource 可以认为是 JNI 中的上下文 Context 
    sp<MediaRecorder> mr = new MediaRecorder(attributionSource);
    ...

    // 创建 JNI JNIMediaRecorderListener , 其收到的消息最终通过 fields.post_event 回到JAVA
    sp<JNIMediaRecorderListener> listener = new JNIMediaRecorderListener(env, thiz, weak_this);
    mr->setListener(listener);
    ...
    // 传递客户端包名，以进行权限跟踪  
    mr->setClientName(clientName);

    // 将创建的 mr 保存到 fields.context，也就是 Java 层 MediaRecorder 中的 mNativeContext  
    setMediaRecorder(env, thiz, mr);
}
```
MediaRecorder JNI 构造  
```cpp
// frameworks/av/media/libmedia/mediarecorder.cpp
MediaRecorder::MediaRecorder(const AttributionSourceState &attributionSource): mSurfaceMediaSource(NULL)
{
    // 通过 binder 获取 MediaPlayerService 
    const sp<IMediaPlayerService> service(getMediaPlayerService());
    if (service != NULL) {
        // 通过 MediaPlayerService 创建 MediaRecorderClient  
        mMediaRecorder = service->createMediaRecorder(attributionSource);
    }
    ...
}
```
MediaRecorder 类关系如下  
```cpp
class MediaRecorder : public BnMediaRecorderClient, public virtual IMediaDeathNotifier {...}
class BnMediaRecorderClient: public BnInterface<IMediaRecorderClient> {...}
```
MediaPlayerService 类关系如下  
```cpp
class MediaPlayerService : public BnMediaPlayerService {...}
class BnMediaPlayerService: public BnInterface<IMediaPlayerService> {...}
```
MediaPlayerService 中 createMediaRecorder 如下  
```cpp
// frameworks/av/media/libmediaplayerservice/MediaPlayerService.cpp
sp<IMediaRecorder> MediaPlayerService::createMediaRecorder(const AttributionSourceState& attributionSource)
{
    ...
    sp<MediaRecorderClient> recorder = new MediaRecorderClient(this, verifiedAttributionSource);
    ...
    return recorder;
}

```
MediaRecorderClient 构造如下  
```cpp
// frameworks/av/media/libmediaplayerservice/MediaRecorderClient.cpp
MediaRecorderClient::MediaRecorderClient(const sp<MediaPlayerService>& service,
        const AttributionSourceState& attributionSource)
{
    // 构造StagefrightRecorder，
    mRecorder = new StagefrightRecorder(attributionSource);
    mMediaPlayerService = service;
}

StagefrightRecorder 类继承关系如下 
struct StagefrightRecorder : public MediaRecorderBase {...}
```
到此 JAVA 层 `new MediaRecord()` 相关源码已经分析完成

## MediaRecorder 参数设置  
Java层 setAudioSource 、 setOutputFormat、setInputSurface 等均调用了 Native 接口，下面以 setAudioSource 为例   
```java
// frameworks/base/media/java/android/media/MediaRecorder.java
public native void setAudioSource(@Source int audioSource) throws IllegalStateException;
```
```cpp
// frameworks/base/media/jni/android_media_MediaRecorder.cpp  
static void android_media_MediaRecorder_setAudioSource(JNIEnv *env, jobject thiz, jint as)
{
    ...
    // 读取初始化时保存的mr
    sp<MediaRecorder> mr = getMediaRecorder(env, thiz);
    if (mr == NULL) {
        jniThrowException(env, "java/lang/IllegalStateException", NULL);
        return;
    }
    process_media_recorder_call(env, mr->setAudioSource(as), "java/lang/RuntimeException", "setAudioSource failed.");
}

// frameworks/av/media/libmedia/mediarecorder.cpp  
status_t MediaRecorder::setVideoSource(int vs)
{
    ...
    // 上面知道，这里的 mMediaRecorder 是 MediaRecorderClient 
    status_t ret = mMediaRecorder->setVideoSource(vs);
    return ret;
}

// frameworks/av/media/libmediaplayerservice/MediaRecorderClient.cpp
status_t MediaRecorderClient::setVideoSource(int vs)
{
    ...
    // 上面知道，这里的 mRecorder 是 StagefrightRecorder 
    return mRecorder->setVideoSource((video_source)vs);
}

// frameworks/av/media/libmediaplayerservice/StagefrightRecorder.cpp
status_t StagefrightRecorder::setVideoSource(video_source vs) {
    ...
    // 最终数据保存在 mVideoSource 中 
    if (vs == VIDEO_SOURCE_DEFAULT) {
        mVideoSource = VIDEO_SOURCE_CAMERA;
    } else {
        mVideoSource = vs;
    }

    return OK;
}
```
同理，其它参数设置多数最终都是保存在 StagefrightRecorder 中，录制相关的流程很多也是在其中控制  

> 本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/134634628> 

## MediaRecorder.prepare 分析
依据前面的分析，最终 prepare 真正实现如下  
```cpp
// frameworks/av/media/libmediaplayerservice/StagefrightRecorder.cpp
status_t StagefrightRecorder::prepareInternal() {
    ...
    status_t status = OK;

    // 依据不同的输出格式，执行不同的 setup 
    switch (mOutputFormat) {
        case OUTPUT_FORMAT_DEFAULT:
        case OUTPUT_FORMAT_THREE_GPP:
        case OUTPUT_FORMAT_MPEG_4:
        case OUTPUT_FORMAT_WEBM:
            status = setupMPEG4orWEBMRecording();
            break;

        case OUTPUT_FORMAT_AMR_NB:
        case OUTPUT_FORMAT_AMR_WB:
            status = setupAMRRecording();
            break;

        case OUTPUT_FORMAT_AAC_ADIF:
        case OUTPUT_FORMAT_AAC_ADTS:
            status = setupAACRecording();
            break;

        case OUTPUT_FORMAT_RTP_AVP:
            status = setupRTPRecording();
            break;

        case OUTPUT_FORMAT_MPEG2TS:
            status = setupMPEG2TSRecording();
            break;

        case OUTPUT_FORMAT_OGG:
            status = setupOggRecording();
            break;

        default:
            ALOGE("Unsupported output file format: %d", mOutputFormat);
            status = UNKNOWN_ERROR;
            break;
    }
    return status;
}
```
这里我们分析一下录制 MP4 的 prepare 流程 setupMPEG4orWEBMRecording   
```cpp
status_t StagefrightRecorder::setupMPEG4orWEBMRecording() {
    // 先清理 MediaWriter 
    mWriter.clear();
    mTotalBitRate = 0;

    status_t err = OK;
    sp<MediaWriter> writer;
    sp<MPEG4Writer> mp4writer;
    if (mOutputFormat == OUTPUT_FORMAT_WEBM) {
        writer = new WebmWriter(mOutputFd);
    } else {
        // 我们这里分析 MP4 录制，MPEG4Writer 主要是用来写入编码后的音视频内容   
        writer = mp4writer = new MPEG4Writer(mOutputFd);
    }

    if (mVideoSource < VIDEO_SOURCE_LIST_END) {
        // 如果编码器未配置，设置默认的编码器  
        setDefaultVideoEncoderIfNecessary();

        sp<MediaSource> mediaSource;
        // 设置 视频源
        err = setupMediaSource(&mediaSource);
        if (err != OK) {
            return err;
        }

        sp<MediaCodecSource> encoder;
        // 编码参数配置，然后通过 MediaCodecSource::Create 创建编码器
        err = setupVideoEncoder(mediaSource, &encoder);
        if (err != OK) {
            return err;
        }

        // MPEG4Writer 添加编码通道，一般会有audio、video 两个，这里是 video
        writer->addSource(encoder);
        mVideoEncoderSource = encoder;
        // 输出文件的码率，是视频和音频总码率之和
        mTotalBitRate += mVideoBitRate;
    }

    // Audio source is added at the end if it exists.
    // This help make sure that the "recoding" sound is suppressed for
    // camcorder applications in the recorded files.
    // disable audio for time lapse recording
    const bool disableAudio = mCaptureFpsEnable && mCaptureFps < mFrameRate;
    if (!disableAudio && mAudioSource != AUDIO_SOURCE_CNT) {
        // 通过  createAudioSource() 配置音频编码参数，进而通过 MediaCodecSource::Create 创建 编码器
        err = setupAudioEncoder(writer);
        
        if (err != OK) return err;
        mTotalBitRate += mAudioBitRate;
    }
    ...
    // 监听 MPEG4Writer 一些参数的回调 
    writer->setListener(mListener);
    mWriter = writer;
    return OK;
}
```
通过如上，可以看到 MediaRecorder.prepare 主要是进行参数的配置、编码器的初始化  

## MediaRecorder.start 分析
依据前面的分析，最终 start 真正实现如下  
```cpp
//  frameworks/av/media/libmediaplayerservice/StagefrightRecorder.cpp
status_t StagefrightRecorder::start() {
    Mutex::Autolock autolock(mLock);
    if (mOutputFd < 0) {
        ALOGE("Output file descriptor is invalid");
        return INVALID_OPERATION;
    }

    status_t status = OK;

    if (mVideoSource != VIDEO_SOURCE_SURFACE) {
        status = prepareInternal();
        if (status != OK) {
            return status;
        }
    }

    switch (mOutputFormat) {
        case OUTPUT_FORMAT_DEFAULT:
        case OUTPUT_FORMAT_THREE_GPP:
        case OUTPUT_FORMAT_MPEG_4:
        case OUTPUT_FORMAT_WEBM:
        {
            sp<MetaData> meta = new MetaData;
            // 设置 meta 信息
            setupMPEG4orWEBMMetaData(&meta);

            // MPEG4Writer 传入 meta 信息，
            // startWriterThread 开启写入线程 
            // setupAndStartLooper 启动 ALooper, mReflector 用于信息传递 
            // 通过 writeFtypBox(MetaData *param) 写入
            // startTracks(MetaData *params) 启动音、视频Track，参见 MPEG4Writer::Track::start
            status = mWriter->start(meta.get());
            break;
        }
        ...
    }

    if (status != OK) {
        // start 异常 
        mWriter.clear();
        mWriter = NULL;
    }

    if ((status == OK) && (!mStarted)) {
        mAnalyticsDirty = true;
        mStarted = true;
        ...
        // 用于编码耗电统计 
        addBatteryData(params);
    }

    return status;
}
```

## MediaRecorder.stop 分析
依据前面的分析，最终 stop 真正实现如下  
```cpp
//  frameworks/av/media/libmediaplayerservice/StagefrightRecorder.cpp
status_t StagefrightRecorder::stop() {
    Mutex::Autolock autolock(mLock);
    status_t err = OK;

    if (mCaptureFpsEnable && mCameraSourceTimeLapse != NULL) {
        // 延时录制，详细可参见 CameraSourceTimeLapse.cpp
        mCameraSourceTimeLapse->startQuickReadReturns();
        mCameraSourceTimeLapse = NULL;
    }

    int64_t stopTimeUs = systemTime() / 1000;
    for (const auto &source : { mAudioEncoderSource, mVideoEncoderSource }) {
        // 设置停止时间戳
        if (source != nullptr && OK != source->setStopTimeUs(stopTimeUs)) {}
    }

    if (mWriter != NULL) {
        // MPEG4Writer 的停止 ，实际调用其  reset(true, true)
        // stopWriterThread() 停止写的线程
        // Track 停止
        // release 关闭文件，停止释放Looper,资源状态重置
        err = mWriter->stop();
        mLastSeqNo = mWriter->getSequenceNum();
        mWriter.clear();
    }

    // 写入参数相关信息
    flushAndResetMetrics(true);

    // 重置参数状态
    mDurationRecordedUs = 0;
    mDurationPausedUs = 0;
    mNPauses = 0;
    mTotalPausedDurationUs = 0;
    mPauseStartTimeUs = 0;
    mStartedRecordingUs = 0;

    mGraphicBufferProducer.clear();
    mPersistentSurface.clear();
    mAudioEncoderSource.clear();
    mVideoEncoderSource.clear();
    ......
    return err;
}
```

## 结语
到这里，已经完成了 MediaRecorder 录制 Framework 源码的分析。    
其它部分流程，可以对照参见 StagefrightRecorder.cpp 中源码。希望对你有所帮助。  
如果你在使用MediaRecorder的过程中遇到了其他问题，欢迎留言讨论。    
如果你觉得本文还不错，可以点赞+收藏。

---

相关文章  
[安卓MediaRecorder(1)录制音频的详细使用](https://h89.cn/archives/77.html)  
[安卓MediaRecorder(2)录制源码分析](https://h89.cn/archives/79.html)  
[安卓MediaRecorder(3)音频采集编码写入源码分析](https://h89.cn/archives/116.html)  
[安卓MediaRecorder(4)视频采集编码写入源码分析](https://h89.cn/archives/124.html)  
