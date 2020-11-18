[TOC]


>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/134599828>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>


## 引言

在Android开发过程中，我们经常需要处理音频或视频相关的功能。比如，我们要做一个简单的录音机或者录像机。  
在Android中录制音频有两种方式：`MediaRecorder`和`AudioRecord`。两者的主要区别在于：  
- `MediaRecorder`提供了一种更高级别的API，能够直接录制并保存为特定的媒体文件格式（如MP3、AAC、AMR等）。其底层实际也使用了`AudioRecord`。  
- `AudioRecord`提供了更底层的API，可以让我们自定义更多关于音频采样率、通道数等参数。但使用起来会比较复杂。  
本文主要介绍如何在Android中使用`MediaRecorder`进行录音，并附带一些常见的问题及其解决方案。  

## 使用 MediaRecorder 的步骤

使用MediaRecorder进行录音和录像，主要有以下几个步骤：

1. 创建一个MediaRecorder对象。
2. 设置MediaRecorder的各种参数，包括音视频源、输出文件、编码格式等等。
3. 调用prepare()方法，让MediaRecorder做好开始录制的准备。
4. 调用start()方法，开始录制。
5. 在合适的时间调用stop()方法，结束录制。
6. 最后别忘了调用release()方法，释放资源。

下面是一个简单的录音示例：
```java
import android.media.MediaRecorder;
import android.os.Environment;

public class RecorderAudio {
    public static final int RECORDER_SAMPLERATE = 44100;
    public static final String AUDIO_RECORDER_FOLDER = "AudioRecorder";

    public static MediaRecorder getRecorder() {
        File dir = new File(Environment.getExternalStorageDirectory(), AUDIO_RECORDER_FOLDER);
        if (!dir.exists()) {
            dir.mkdirs();
        }
        File file = new File(dir, System.currentTimeMillis() + ".amr");

        // 创建一个MediaRecorder对象
        MediaRecorder recorder = new MediaRecorder();

        // 设置音频源为麦克风
        recorder.setAudioSource(MediaRecorder.AudioSource.MIC);
        // 设置音频输出格式
        recorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
        // 设置音频编码格式
        recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
        // 设置输出文件路径
        recorder.setOutputFile(file.getAbsolutePath());

        return recorder;
    }

    public static void prepare(MediaRecorder recorder) {
        try {
            // 准备录制，初始化 MediaRecorder 的各种状态，并根据配置的信息创建一个 MediaCodec 对象。
            recorder.prepare();
        } catch (IOException e) {
            Log.e("MediaRecorder", "prepare() failed");
        }
    }

    public static void start(MediaRecorder recorder) {
        try {
            // 开始录制，开始真正的录音工作。它会启动一个循环来从 MediaCodec 对象中取出编码后的音频数据，然后写入到指定的文件中。
            recorder.start();
        } catch (RuntimeException e) {
            Log.e("MediaRecorder", "start() failed");
        }
    }

    public static void stop(MediaRecorder recorder) {
        // 停止录制，会让循环停止，并等待剩余的数据全部写入文件
        recorder.stop();
        // 释放掉所有的资源，包括`MediaRecorder`对象自身。
        recorder.release();
        recorder = null;
    }
}
```
**注意**：要确保所有的操作都在同一个线程中执行，否则可能会导致崩溃或异常。


如果想要在录制过程中，查看音量大小，可以通过调用`getMaxAmplitude()`方法来获取一小段时间内音频源数据中的最大振幅。  
例如，每秒调用一次，可以得到近似音量值：  
`int maxAmplitude = recorder.getMaxAmplitude();`    

## 常见问题及解决思路

在实际使用MediaRecorder的过程中，可能会遇到各种各样的问题。这里列举一些常见的问题以及解决思路：

### 无法访问存储卡目录

**原因**：Android 6.0及以上版本要求用户明确授予应用读取外部存储的权限。

**解决**：在运行以上代码之前，需要动态申请权限：

```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) 
        != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, 
        new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, REQUEST_WRITE_STORAGE);
}
```
并在`onRequestPermissionsResult`回调方法中处理结果：

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    if (requestCode == REQUEST_WRITE_STORAGE && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        startRecording();
    } else {
        Toast.makeText(this, "No storage permission", Toast.LENGTH_SHORT).show();
    }
}
```

### 录制的音频文件没有声音

**原因**：这可能是由于设置的音频源或编码器不正确导致的。

**解决**：确认已经设置了正确的音频源和编码器，并且麦克风功能正常。

### 录制过程中出现异常

**原因**：这可能是由于在多线程环境下操作`MediaRecorder`导致的。

**解决**：确保所有对`MediaRecorder`的操作都在同一个线程中执行。


### MediaRecorder无法正常启动

这是最常见的问题。如果MediaRecorder无法正常启动，可能是由于参数设置错误，或者是设备不支持某种格式。
解决这个问题的方法是检查所有的参数设置，确保它们都是正确的，并且符合设备的要求。

### 录制的音质或者画质很差

这可能是因为采样率或者比特率设置得太低。解决这个问题的方法是提高采样率或者比特率，但是这样会增加文件的大小。

### 录制的文件无法播放

这可能是因为编码格式设置得不正确，或者设备不支持这种格式。解决这个问题的方法是更换编码格式，选择一种设备支持的格式。

## 结语

以上就是关于Android中MediaRecorder录制音频的详细使用的一些介绍。希望对你有所帮助。  
如果你在使用MediaRecorder的过程中遇到了其他问题，欢迎留言讨论。  
如果你觉得本文还不错，可以点赞+收藏。  


--- 

相关文章  
[安卓MediaRecorder(1)录制音频的详细使用](https://h89.cn/archives/77.html)  
[安卓MediaRecorder(2)录制源码分析](https://h89.cn/archives/79.html)  
[安卓MediaRecorder(3)音频采集编码写入源码分析](https://h89.cn/archives/116.html)  
[安卓MediaRecorder(4)视频采集编码写入源码分析](https://h89.cn/archives/124.html)  


