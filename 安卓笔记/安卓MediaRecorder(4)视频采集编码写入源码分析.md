
[TOC]

>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/134896821>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>

## 前言
通过 [文2](https://h89.cn/archives/79.html) 我们知道了 MediaRecorder 各个接口 Framework 中的实现。  
通过 [文3](https://h89.cn/archives/116.html) 我们 知道了 MediaRecorder 底层音频的采集、编码、写入文件等详细流程。  
本文主要介绍 MediaRecorder 视频的采集、编码等相关流程。

## 视频采集
在[ 文1 ](https://h89.cn/archives/77.html)我们知道了如何使用 MediaRecorder 录制音频，那么如何同时录制声音和视频呢，可以参见 [Demo Camera2Video][]，这里不再贴代码。  
通过此示例，我们知道录制视频需要如下设置  
```kotlin
val surface = MediaCodec.createPersistentInputSurface()
...
setOutputFormat(MediaRecorder.OutputFormat.MPEG_4)
setVideoSource(MediaRecorder.VideoSource.SURFACE)
setVideoEncoder(videoEncoder)
setInputSurface(surface)
```
也就是视频采集源，就是这个 `surface = MediaCodec.createPersistentInputSurface()`。
也就是 `mInputSurface`，也是 PreviewFragment 中 encoderSurface，最终传递到 eglEncoderSurface  
`eglEncoderSurface = EGL14.eglCreateWindowSurface(eglDisplay, eglConfig, encoderSurface, surfaceAttribs, 0)`  
在 [HardwarePipeline](https://gitee.com/chenjim/CameraDemo/blob/master/Camera2Video/app/src/main/java/com/example/android/camera2/video/HardwarePipeline.kt)中，可以看到添加了一路预览流：  
```kotlin
    // 创建一个 GL_TEXTURE_EXTERNAL_OES 纹理
    cameraTexId = createTexture()
    cameraTexture = SurfaceTexture(cameraTexId)
    ...
    session.device.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW).apply {
        // 添加预览 surface target
        addTarget(cameraSurface)
    }.build()
```
基于 `cameraTexture.setOnFrameAvailableListener(this)`, 当预览流可用时，会进行如下消费、复制操作：  
```kotlin
private fun onFrameAvailableImpl(surfaceTexture: SurfaceTexture) {
    if (eglContext == EGL14.EGL_NO_CONTEXT) {
        return
    }

    /* 消费掉camera出来的流 ，updateTexImage 相关介绍可以参考 https://blog.csdn.net/CSqingchen/article/details/135637088 */
    cameraTexture.updateTexImage()

    /** 复制 cameraTexture 到 eglRenderSurface  */
    if (eglRenderSurface != EGL_NO_SURFACE) {
        copyCameraToRender()
    }

    /** 复制 eglRenderSurface 到 TextureView 显示*/
    copyRenderToPreview()

    /**  复制 eglRenderSurface 到 eglEncoderSurface ，通过消息 encoder.frameAvailable() 告知编码*/
    if (eglEncoderSurface != EGL_NO_SURFACE && currentlyRecording) {
        copyRenderToEncode()
    }
}
```
至此，我们知道了 MediaRecorder 采集视频的数据流。  
这里是基于 [Demo Camera2Video][] 分析，其它情况也差不多。

## 视频编码
在示例 Camera2Video 中，如果 useMediaRecorder 为 false，编码相关代码如下：  
```kotlin
public fun drainEncoder(): Boolean {
    while (true) {
        // 编码 
        var encoderStatus: Int = mEncoder.dequeueOutputBuffer(mBufferInfo, TIMEOUT_USEC)
        ...
        // 取编码的数据
        var encodedData: ByteBuffer? = mEncoder.getOutputBuffer(encoderStatus)
        ...
        //写入编码后数据到文件
        mMuxer.writeSampleData(mVideoTrack, encodedData, mBufferInfo)
    }
}
```
完整代码参见 [EncoderWrapper.kt](https://gitee.com/chenjim/CameraDemo/blob/master/Camera2Video/app/src/main/java/com/example/android/camera2/video/EncoderWrapper.kt)   
如果 useMediaRecorder 为 true ，编码及写入均在Framework，我们可以从 setInputSurface 往下底层查看。  
通过 [文2](https://h89.cn/archives/79.html) ,setInputSurface的最终实现如下  
```cpp
status_t StagefrightRecorder::setInputSurface(
        const sp<PersistentSurface>& surface) {
    mPersistentSurface = surface;
    return OK;
}
```
编码器初始化创建如下：  
```cpp
status_t StagefrightRecorder::setupVideoEncoder(const sp<MediaSource> &cameraSource,
        sp<MediaCodecSource> *source){
    ...
    sp<MediaCodecSource> encoder = MediaCodecSource::Create(
        mLooper, format, cameraSource, mPersistentSurface, flags);
    ...
}
status_t MediaCodecSource::initEncoder() {
    ...
    if (mPersistentSurface != NULL) {
        err = mEncoder->setInputSurface(mPersistentSurface);
    }
    ...
}
status_t MediaCodec::setInputSurface(
        const sp<PersistentSurface> &surface) {
    sp<AMessage> msg = new AMessage(kWhatSetInputSurface, this);
    msg->setObject("input-surface", surface.get());
    sp<AMessage> response;
    return PostAndAwaitResponse(msg, &response);
}
```
通过如下代码将 Camera 数据源设置为编码
```cpp
status_t StagefrightRecorder::setupCameraSource(sp<CameraSource> *cameraSource) {

    if (mCaptureFpsEnable) {
        *cameraSource = CameraSource::CreateFromCamera(
                mCamera, mCameraProxy, mCameraId, clientName, uid, pid,
                videoSize, mFrameRate,
                mPreviewSurface);
    }
}
...
setupVideoEncoder(mediaSource, &encoder);
```

## 视频编码写入
视频的编码写入流程同 [文3](https://h89.cn/archives/116.html) 音频的编码、写入。


## 结语 

到这里，通过相关文章的介绍，我们已经很清晰 MediaRecorder 底层的音、视频采集、编码、写入编码后内容等相关流程的源码实现。
希望对你有所帮助。如果你在使用 MediaRecorder 的过程中遇到了其他问题，欢迎留言讨论。    
如果你觉得本文还不错，可以点赞+收藏。


---
相关文章  
[安卓MediaRecorder(1)录制音频的详细使用](https://h89.cn/archives/77.html)  
[安卓MediaRecorder(2)录制源码分析](https://h89.cn/archives/79.html)  
[安卓MediaRecorder(3)音频采集编码写入源码分析](https://h89.cn/archives/116.html)  
[安卓MediaRecorder(4)视频采集编码写入源码分析](https://h89.cn/archives/124.html)  



[Demo Camera2Video]:https://gitee.com/chenjim/CameraDemo/tree/master/Camera2Video

