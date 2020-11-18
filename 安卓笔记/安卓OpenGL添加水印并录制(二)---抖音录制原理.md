
[TOC]

> 本文首发地址 <https://h89.cn/archives/146.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  
> 源码地址: [Gitee: OpenGLRecorder](https://gitee.com/chenjim/CameraDemo/tree/master/OpenGLRecorder)


通过 [前文](https://chenjim.blog.csdn.net/article/details/105492716) 我们知道了如何采集 Camera 视频，叠加水印、贴纸保存为MP4，但是录制视频并没有音频，本文进一步介绍添加音频录制实现。

## 前文回顾
前文介绍的视频处理流程及主要类如下    
1. [CameraGlView](https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/CameraGlView.java) 用来显示Camera预览的View  
   `CameraGlView extends GLSurfaceView`  
2. 创建 SurfaceTexture 来显示 Camera 预览，参见[ CameraRenderer.java](https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/CameraRenderer.java)  
    ```java
    mSurfaceTexture = new SurfaceTexture(mTextures[0]);  
    mCameraHelper.startPreview(mSurfaceTexture);
    ```
3. [CameraFilter.java](https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/filter/CameraFilter.java)，通过 OpenGL 将 Camera 数据写入 FBO(Frame Buffer Object 帧缓存)
4. [TimeFilter.java](https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/filter/TimeFilter.java)，通过 OpenGL 在 FBO 上添加时间水印。可以参考此处添加贴纸、美颜等。  
5. [ScreenFilter.java](https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/filter/ScreenFilter.java)，将 FBO 绘制到 mSurfaceTexture  
6. [MediaRecorder.java][MediaRecorder.link]，用 MediaCodec 和 EGL 对 Surface 的内容采集编码为 avc 并写入到 MP4 文件  

## 音频处理
音频处理主要分为采集、编码、写入几个阶段，对应的实现如下所述  
1. 我们可以使用 AudioRecord 采集音频，具体实现参见 [AudioRecordPcm.kt](https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/audio/AudioRecordPcm.kt)  
2. 对采集到的 PCM 帧通过 MediaCodec 编码为 AAC ，具体实现参见 [PcmEncodeAacCtrl.kt](https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/audio/PcmEncodeAacCtrl.kt)  
3. 将编码后的音频写入到 MP4 文件，这里只需增加 AudioTrack ，然后 writeSampleData 即可，具体实现参见 [MediaRecorder.java][MediaRecorder.link]  

注意时序问题，在快速极端操作，可能会有异常，如果你遇到，欢迎留言讨论解决。

## 留个小思考 
在当前代码中存在 openGL 泄露 , 通过 perfetto 分析结果如下  
![](https://pic.chenjim.com/202402112102205.png-blog)   
使用 perfetto 分析 GL 泄露的方法，可以参见 [Android性能优化--Perfetto分析native内存泄露](https://h89.cn/archives/24.html)   
解决结果参见 [issues](https://gitee.com/chenjim/CameraDemo/issues/I7U5OL)  

## 总结

综上所述，即为抖音视频录制的基本实现，整个过程的数据流如下图    
![](https://pic.chenjim.com/202402092158380.png-blog) 
希望本文对你有所帮助，若使用遇到问题，可以留言讨论。  

---

参考文章:
- [OpenGL ES SDK for Android: High Quality Text Rendering](https://arm-software.github.io/opengl-es-sdk-for-android/high_quality_text.html)
- [Github：opengl-es-sdk-for-android/HighQualityTextJava](https://github.com/ARM-software/opengl-es-sdk-for-android/tree/master/samples/advanced_samples/HighQualityTextJava)
- [Github：android-openGL-canvas](https://github.com/ChillingVan/android-openGL-canvas)
- [Github:Media for Mobile is a set of easy to use components and API for a wide range of media scenarios such as video editing and capturing](https://github.com/INDExOS/media-for-mobile)
- [VideoRecorder高性能任意尺寸视频录制 断点录制 离屏录制 ](https://github.com/lll-01/VideoRecorder) 录制时的Canvas API支持 实时滤镜，[相关资料介绍Link](https://github.com/lll-01/VideoRecorder/blob/master/app/doc/%E7%9B%B8%E5%85%B3%E8%B5%84%E6%96%99.md)
- [抖音录制视频预习资料](https://www.jianshu.com/p/6aaa44072272)


---

相关文章  
[安卓MediaRecorder(1)录制音频的详细使用](https://h89.cn/archives/77.html)  
[安卓MediaRecorder(2)录制源码分析](https://h89.cn/archives/79.html)  
[安卓MediaRecorder(3)音频采集编码写入源码分析](https://h89.cn/archives/116.html)  
[安卓MediaRecorder(4)视频采集编码写入源码分析](https://h89.cn/archives/124.html)  

---

[MediaRecorder.link]:https://gitee.com/chenjim/CameraDemo/blob/master/OpenGLRecorder/app/src/main/java/com/chenjim/glrecorder/MediaRecorder.jav