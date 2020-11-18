
[TOC]

> 本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/43817975>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>

### 软件修改Bitmap像素点实现模糊
此方法无sdk版本要求，但是效率非常非常差，没有可使用性。  
具体源码可以参考后文附的完整实例  

### 使用 RenderScript 模糊
ScriptIntrinsicBlur 要求 android sdk 版本最低 17, Android 12(API 31) 开始废弃   
设备和组件制造商已停止提供硬件加速支持，预计将在未来的版本中完全取消对 RenderScript 的支持。  

详细使用代码如下：  
```java
/**
 * 通过调用系统高斯模糊api的方法模糊
 *
 * @param bitmap    source bitmap
 * @param outBitmap out bitmap
 * @param radius    0 < radius <= 25
 * @param context   context
 * @return out bitmap
 */
public static Bitmap blurBitmap(Bitmap bitmap, Bitmap outBitmap, float radius, Context context) {
    //Let's create an empty bitmap with the same size of the bitmap we want to blur
    //Bitmap outBitmap = Bitmap.createBitmap(bitmap.getWidth(), bitmap.getHeight(), Config.ARGB_8888);

    //Instantiate a new Renderscript
    RenderScript rs = RenderScript.create(context);

    //Create an Intrinsic Blur Script using the Renderscript
    ScriptIntrinsicBlur blurScript = ScriptIntrinsicBlur.create(rs, Element.U8_4(rs));

    //Create the Allocations (in/out) with the Renderscript and the in/out bitmaps
    Allocation allIn = Allocation.createFromBitmap(rs, bitmap);
    Allocation allOut = Allocation.createFromBitmap(rs, outBitmap);

    //Set the radius of the blur
    blurScript.setRadius(radius);

    //Perform the Renderscript
    blurScript.setInput(allIn);
    blurScript.forEach(allOut);

    //Copy the final bitmap created by the out Allocation to the outBitmap
    allOut.copyTo(outBitmap);

    //recycle the original bitmap
//   bitmap.recycle();

    //After finishing everything, we destroy the Renderscript.
    rs.destroy();

    return outBitmap;
}
```

### 使用 Toolkit 模糊  
Toolkit 是 ScriptIntrinsicBlur 的替代方案之一，效率更高   
Toolkit.kt 只是一个 JAVA 接口，主要实现是在JNI中  
以模糊为例，其部分实现代码如下 ，参见 Blur.cpp  
```cpp
/**
 * Full blur of a line of U_8 data.
 *
 * @param outPtr Where to store the results
 * @param xstart The index of the section we're starting to blur.
 * @param xend  The end index of the section.
 * @param currentY The index of the line we're blurring.
 */
void BlurTask::kernelU1(void *outPtr, uint32_t xstart, uint32_t xend, uint32_t currentY) {
    float buf[4 * 2048];
    const uint32_t stride = mSizeX * mVectorSize;

    uchar *out = (uchar *)outPtr;
    uint32_t x1 = xstart;
    uint32_t x2 = xend;

    float *fout = (float *)buf;
    int y = currentY;
    if ((y > mIradius) && (y < ((int)mSizeY - mIradius -1))) {
        const uchar *pi = mIn + (y - mIradius) * stride;
        OneVFU1(fout, pi, stride, mFp, mIradius * 2 + 1, mSizeX, mUsesSimd);
    } else {
        x1 = 0;
        while(mSizeX > x1) {
            OneVU1(mSizeY, fout, x1, y, mIn, stride, mFp, mIradius);
            fout++;
            x1++;
        }
    }

    x1 = xstart;
    while ((x1 < x2) &&
           ((x1 < (uint32_t)mIradius) || (((uintptr_t)out) & 0x3))) {
        OneHU1(mSizeX, out, x1, buf, mFp, mIradius);
        out++;
        x1++;
    }

    while(x2 > x1) {
        OneHU1(mSizeX, out, x1, buf, mFp, mIradius);
        out++;
        x1++;
    }
}
```
只需要下载 renderscript-intrinsics-replacement-toolkit ，编译 renderscript-toolkit 模块 ，引用即可   
也可直接下载已经编译好的文件 [renderscript-toolkit-debug-1.0.0.aar](https://gitee.com/chenjim/android-blur/raw/main/app/libs/renderscript-toolkit-debug-1.0.0.aar) 引用     
```gradle
implementation (project(":renderscript-toolkit"))   
//或者如下方式，引入下载的aar
implementation (files("libs/renderscript-toolkit-debug-1.0.0.aar"))
```
使用方法如下  
```java
Toolkit.blur(inputBitmap, radius, null)
```

### Toolkit 和 RenderScript 效率对比  
结果如下，可以看到 Toolkit 在部分场景已经超越 RenderScript，特别是在处理小图时  
```
D  FastBlur 1898X3069 cost time: 1055
D  RSBlur 1898X3069 cost time: 42
D  Toolkit 1898X3069 cost time: 29
```

----

以上内容源码地址  <https://gitee.com/chenjim/android-blur/>  


Toolkit 是不是最终的方案呢？
如果对性能有更高的要求，还[可以使用 Vulkan、RenderEffect 继续优化](https://blog.csdn.net/CSqingchen/article/details/134656140)   

本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/43817975>  

### Bitmap缩放处理
对于模糊处理，bitmap尺寸越小，处理的越迅速。  
为了达到需要的模糊效果，通常我们需要对输入 bitmap 缩放的处理，缩放代码如下：
```java
/**
 * 比例压缩图片
 *
 * @param sourceBitmap 源bitmap
 * @param scaleFactor  大于1，将bitmap缩小
 * @return 缩小scaleFactor倍后的bitmap
 */
public static Bitmap compressBitmap(Bitmap sourceBitmap, float scaleFactor) {
    Bitmap overlay = Bitmap.createBitmap((int) (sourceBitmap.getWidth() / scaleFactor),
            (int) (sourceBitmap.getHeight() / scaleFactor), Config.ARGB_8888);
    Canvas canvas = new Canvas(overlay);
    canvas.translate(0, 0);
    canvas.scale(1 / scaleFactor, 1 / scaleFactor);
    Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG | Paint.FILTER_BITMAP_FLAG);
    canvas.drawBitmap(sourceBitmap, 0, 0, paint);
    return overlay;
}
```


### 改变图片的对比度/明暗度
详细说明见代码注释，示例代码
```java
/**
 * 改变图片对比度,达到使图片明暗变化的效果
 *
 * @param srcBitmap source bitmap
 * @param contrast  图片亮度，0：全黑；小于1，比原图暗；1.0f原图；大于1比原图亮
 * @return bitmap
 */
public static Bitmap darkBitmap(Bitmap srcBitmap, float contrast) {

    float offset = (float) 0.0; //picture RGB offset

    int imgHeight, imgWidth;
    imgHeight = srcBitmap.getHeight();
    imgWidth = srcBitmap.getWidth();

    Bitmap bmp = Bitmap.createBitmap(imgWidth, imgHeight, Config.ARGB_8888);
    ColorMatrix cMatrix = new ColorMatrix();
    cMatrix.set(new float[]{contrast, 0, 0, 0, offset,
            0, contrast, 0, 0, offset,
            0, 0, contrast, 0, offset,
            0, 0, 0, 1, 0});

    Paint paint = new Paint();
    paint.setColorFilter(new ColorMatrixColorFilter(cMatrix));

    Canvas canvas = new Canvas(bmp);
    canvas.drawBitmap(srcBitmap, 0, 0, paint);

    return bmp;
}
```

---

以上就是基于 ScriptIntrinsicBlur 和 Toolkit 对 Bitmap 模糊实现，希望对你有所帮助。  
如果你在使用过程遇到问题，可以留言讨论。  
如果你觉得本文写的还不错，欢迎点赞+收藏。  

---

**相关文章**  
[Android Bitmap 使用 ScriptIntrinsicBlur、Toolkit 实现模糊](https://blog.csdn.net/CSqingchen/article/details/43817975)  
[Android Bitmap 使用Vukan、RenderEffect、GLSL实现模糊)](https://blog.csdn.net/CSqingchen/article/details/134656140)  