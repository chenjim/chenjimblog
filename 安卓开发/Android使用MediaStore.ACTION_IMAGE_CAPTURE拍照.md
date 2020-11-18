
[TOC]

> 本文首发地址 <https://h89.cn/archives/187.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  
> 本文示例地址 <https://gitee.com/chenjim/CaptureIntent>

## 用 ACTION_IMAGE_CAPTURE 拍照
```kotlin
private fun startTakePhoto() {
    val intent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
    if (intent.resolveActivity(packageManager) != null) {
        val contentValues = ContentValues(2)

        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N && false) {
            // 新版本 不一定支持
            contentValues.put(MediaStore.Images.Media.DATA, createImageFile().path)
        } else {
            // 拍完保存在Pictures，这里并不需要写文件权限，后续读取是否需要最好加上
            contentValues.put(MediaStore.Images.Media.DISPLAY_NAME, createImageFileName())
        }

        contentValues.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
        mPhotoUri = contentResolver.insert(
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
            contentValues
        )
        intent.putExtra(MediaStore.EXTRA_OUTPUT, mPhotoUri)

        startActivityForResult(intent, CAPTURE_IMG_RQ)
    } else {
        Log.e(TAG, "no activity handle $intent")
    }
}
```
以上在 MIUI 验证通过，若其它手机存在异常，欢迎留言讨论。  
这里是否可以调用三方相机，比如美图秀秀呢？   
在 美图秀秀 安卓应用 AndroidManifest.xml 中，我们可以看到如下配置，也就是支持其它应用调用的  
```xml
<activity android:name="com.mt.mtxx.camera.view.CameraActivity">
    <intent-filter>
        <action android:name="android.media.action.IMAGE_CAPTURE"/>
        <action android:name="com.meitu.ble.intent.capture_with_rc"/>
        <action android:name="com.meitu.mtxx.action.image_capture"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>    
```
但是从安卓 11 开始，无法调用三方相机(比如美图秀秀)拍摄，具体可以参见 [博文](https://zhuanlan.zhihu.com/p/189697315)    
## 加载拍摄照片
```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == CAPTURE_IMG_RQ) {
        if (resultCode == RESULT_OK) {
            onImageCaptured()
        } else {
            Log.w(TAG, "on fail:$resultCode")
        }
    }
}

private fun onImageCaptured() {
    val projection = arrayOf(
        MediaStore.MediaColumns._ID,
        MediaStore.Images.ImageColumns.ORIENTATION,
        MediaStore.Images.Media.DATA
    )
    val cursor = contentResolver.query(mPhotoUri!!, projection, null, null, null)
    cursor?.run {
        cursor.moveToFirst()
        try {
            val path = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA))
            Log.d(TAG, "out file path:$path")
            if (File(path).exists()) {
                Glide.with(this@MainActivity).load(path).into(findViewById(R.id.image_view))

                //也可以直接用上面的Uri
//                    Glide.with(this@MainActivity).load(mPhotoUri).into(findViewById(R.id.image_view))
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
        cursor.close()
    }
}
```
如果想采用 ActivityResultContracts.TakePicture 拍摄，可参考后文拍摄视频相关代码

---

## 用 ActivityResultContracts 拍摄视频
注意这里会用到 [FileProvider](https://developer.android.com/reference/androidx/core/content/FileProvider) 相关知识
AndroidManifest.xml 中需要添加以下内容
```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="com.chenjim.captureintent.provider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```
`res\xml\file_paths.xml`内容如下
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <root-path name="root" path="" />
    <files-path name="files" path="files" />
    <cache-path name="cache" path="cache" />
    <external-path name="external" path="external" />
    <external-files-path name="name" path="path" />
    <external-cache-path name="name" path="path" />
</paths>
```
使用ActivityResultContracts.CaptureVideo拍摄视频
```kotlin
private var mVideoUrl: Uri? = null
private val takeVideo = registerForActivityResult(ActivityResultContracts.CaptureVideo()) { result ->
    if (result == true) {
        Log.d(TAG, "success")
        Glide.with(this@MainActivity).load(mVideoUrl).into(findViewById(R.id.video_view))

        //无法像mPhotoUri 一样通过Uri查询到文件路径，如果需要，可以重新导出Uri视频资源
        contentResolver.openInputStream(mVideoUrl!!)?.let { stream ->
            val outFile = File(getExternalFilesDir(null), "capture.video.mp4")
            writeFile(stream, outFile)
            Log.d(TAG, "new out path:$outFile")
        }
    } else {
        Log.d(TAG, "fail")
    }
}
private fun startTakeVideo() {
    val file = File(filesDir, "${SimpleDateFormat("yyyyMMdd_HHmmss").format(Date())}.mp4")
    mVideoUrl = FileProvider.getUriForFile(this, applicationContext.packageName + ".provider", file)
    takeVideo.launch(mVideoUrl)
}

fun writeFile(inputStream: InputStream, out: File) {
    val parentDir = out.parentFile ?: return
    if (!parentDir.exists()) {
        parentDir.mkdirs()
    }
    try {
        FileOutputStream(out).use { fileOutputStream ->
            val buf = ByteArray(1024)
            var length: Int
            while (inputStream.read(buf).also { length = it } != -1) {
                fileOutputStream.write(buf, 0, length)
                fileOutputStream.flush()
            }
            inputStream.close()
            fileOutputStream.flush()
        }
    } catch (e: Exception) {
        Log.e(TAG, out.path + " write fail:" + e)
    }
}
```

---

## 常见问题及处理
1. 拍了一张照片,查Media数据库却查不到,可以用如下方法更新Media数据库后查询
> 调用系统的媒体扫描器，可以将您的照片添加到媒体提供商的数据库中，
> 使 Android 图库应用中显示这些照片，并使它们可供其他应用使用。
```java
final String[] SCAN_TYPES = {"image/jpeg"};
MediaScannerConnection.scanFile(mContext, new String[]{file.getPath()}, SCAN_TYPES, null);
```
或者  
```kotlin
 Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE).also { mediaScanIntent ->
     mediaScanIntent.data = Uri.fromFile(File(imagePath))
     sendBroadcast(mediaScanIntent)
 }
```

---
相关文章 
- [Android Developer Training Camera Photo](https://developer.android.com/training/camera/photobasics)
- [Android 10 适配要点，作用域存储](https://blog.csdn.net/guolin_blog/article/details/105419420)
- [stackoverflow@android-action-image-capture-intent](http://stackoverflow.com/questions/1910608/android-action-image-capture-intent)
- <https://github.com/deepwinter/AndroidCameraTester>
- [Android 7.0 行为变更](https://developer.android.google.cn/about/versions/nougat/android-7.0-changes) 以及 [通过FileProvider在应用间共享文件](https://blog.csdn.net/lmj623565791/article/details/72859156)  