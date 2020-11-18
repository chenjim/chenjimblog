
[TOC]

>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/134656140>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>

通过 [Android Bitmap 使用 ScriptIntrinsicBlur、Toolkit 实现模糊](https://blog.csdn.net/CSqingchen/article/details/43817975)，我们已经知道两种实现模糊方法。  
本文主要讲解另外几种高效实现Bitmap模糊的方法。  


### 使用 RenderEffect 模糊 
RenderEffect 是 Android 中一种用于实现图像特效的类，**最低 API 要求 31** 。  
它允许开发者在不修改原始图像数据的情况下，对图像进行各种处理，例如模糊、光晕、阴影等  
对 Bitmap 模糊及注释代码如下   
```kotlin
fun blur(bitmap:Bitmap, radius: Float, outputIndex: Int): Bitmap {

     // 配置跟 bitmap 同样大小的 ImageReader
    val imageReader = ImageReader.newInstance(
        bitmap.width, bitmap.height,
        PixelFormat.RGBA_8888, numberOfOutputImages,
        HardwareBuffer.USAGE_GPU_SAMPLED_IMAGE or HardwareBuffer.USAGE_GPU_COLOR_OUTPUT
    )
    val renderNode = RenderNode("RenderEffect")
    val hardwareRenderer = HardwareRenderer()

    // 将 ImageReader 的surface 设置到 HardwareRenderer 中
    hardwareRenderer.setSurface(imageReader.surface)
    hardwareRenderer.setContentRoot(renderNode)
    renderNode.setPosition(0, 0, imageReader.width, imageReader.height)

    // 使用 RenderEffect 配置模糊效果，并设置到 RenderNode 中。  
    val blurRenderEffect = RenderEffect.createBlurEffect(
        radius, radius,
        Shader.TileMode.MIRROR
    )
    renderNode.setRenderEffect(renderEffect)

    // 通过 RenderNode 的 RenderCanvas 绘制 Bitmap。   
    val renderCanvas = renderNode.beginRecording()
    renderCanvas.drawBitmap(bitmap, 0f, 0f, null)
    renderNode.endRecording()

    // 通过 HardwareRenderer 创建 Render 异步请求。   
    hardwareRenderer.createRenderRequest()
        .setWaitForPresent(true)
        .syncAndDraw()

    // 通过 ImageReader 获取模糊后的 Image 。
    val image = imageReader.acquireNextImage() ?: throw RuntimeException("No Image")

    // 将 Image 的 HardwareBuffer 包装为 Bitmap , 也就是模糊后的。   
    val hardwareBuffer = image.hardwareBuffer ?: throw RuntimeException("No HardwareBuffer")
    val bitmap = Bitmap.wrapHardwareBuffer(hardwareBuffer, null)
        ?: throw RuntimeException("Create Bitmap Failed")
    hardwareBuffer.close()
    image.close()
    return bitmap
}

```
完整实例参考 RenderEffectImageProcessor.kt  
还可以通过设置 RenderEffect 的其他属性，如 setColorFilter( )方法，为模糊后的 Bitmap 添加颜色滤镜。   

### 使用 Vukan 模糊
Vulkan 是一种低开销、跨平台的 API，用于高性能 3D 图形。  
Android平台包含 Khronos Group 的 Vulkan API规范的特定实现。  
使用 Vukan 模糊的核心代码如下，可参考 ImageProcessor.cpp  
```cpp
bool ImageProcessor::blur(float radius, int outputIndex) {
    RET_CHECK(1.0f <= radius && radius <= 25.0f);

    //高斯模糊配置，在后文 GLSL 同样 适用
    constexpr float e = 2.718281828459045f;
    constexpr float pi = 3.1415926535897932f;
    float sigma = 0.4f * radius + 0.6f;
    float coeff1 = 1.0f / (std::sqrtf(2.0f * pi) * sigma);
    float coeff2 = -1.0f / (2.0f * sigma * sigma);
    int32_t iRadius = static_cast<int>(std::ceilf(radius));
    float normalizeFactor = 0.0f;
    for (int r = -iRadius; r <= iRadius; r++) {
        const float value = coeff1 * std::powf(e, coeff2 * static_cast<float>(r * r));
        mBlurData.kernel[r + iRadius] = value;
        normalizeFactor += value;
    }
    normalizeFactor = 1.0f / normalizeFactor;
    for (int r = -iRadius; r <= iRadius; r++) {
        mBlurData.kernel[r + iRadius] *= normalizeFactor;
    }
    RET_CHECK(mBlurUniformBuffer->copyFrom(&mBlurData));

    // 应用两阶段模糊算法:一个水平模糊核，然后是一个垂直模糊核。
    // 比单遍应用一个2D模糊滤镜更高效。
    // 两遍模糊算法有两个核，每个核的时间复杂度为O(半径)，
    // 而单遍模糊算法只有一个核，但时间复杂度为O(半径^2)。
    auto cmd = mCommandBuffer->handle();
    RET_CHECK(beginOneTimeCommandBuffer(cmd));

    // 临时映像在第一遍中用作输出存储映像
    mTempImage->recordLayoutTransitionBarrier(cmd, VK_IMAGE_LAYOUT_GENERAL, /*preserveData=*/false);

    // 水平方向高斯模糊
    mBlurHorizontalPipeline->recordComputeCommands(cmd, &iRadius, *mInputImage, *mTempImage,
                                                   mBlurUniformBuffer.get());

    // 临时图像在第二遍中用作输入采样图像，
    // 过渡图像用作输出存储映像。
    mTempImage->recordLayoutTransitionBarrier(cmd, VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL);
    mStagingOutputImage->recordLayoutTransitionBarrier(cmd, VK_IMAGE_LAYOUT_GENERAL,
                                                       /*preserveData=*/false);

    // 数值方向高斯模糊
    mBlurVerticalPipeline->recordComputeCommands(cmd, &iRadius, *mTempImage, *mStagingOutputImage,
                                                 mBlurUniformBuffer.get());

    // 准备将图像从过渡图像复制到输出图像。
    mStagingOutputImage->recordLayoutTransitionBarrier(cmd, VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL);

    // 复制暂存图像到输出图像。
    recordImageCopyingCommand(cmd, *mStagingOutputImage, *mOutputImages[outputIndex]);

    // 提交到队列。
    RET_CHECK(endAndSubmitCommandBuffer(cmd, mContext->queue()));
    return true;
}
```

VulkanContext.cpp 主要是一些初始化   
![](https://pic.chenjim.com/202311301058605.png-blog)  

VulkanResources.cpp 主要是 Buffer 和 Image 的一些封装方法   
![](https://pic.chenjim.com/202311301059379.png-blog)


上层接口参见 VulkanImageProcessor.kt  

### 使用 GLSL 模糊  
主要流程：  
- 将输入 Bitmap 转为纹理  
    GLES31.glTexStorage2D(GLES31.GL_TEXTURE_2D, 1, GLES31.GL_RGBA8, mInputImage.width, mInputImage.height )  
    GLUtils.texImage2D(GLES31.GL_TEXTURE_2D, 0, mInputImage, 0) 
- 通过 OpenGL 处理纹理，同样有水平、竖直模糊，即 mBlurHorizontalProgram 和 mBlurVerticalProgram  
- 将纹理转换为 Bitmap ，即 copyPixelsToHardwareBuffer ，这里也是耗时最多的  

完整实例参考 GLSLImageProcessor.kt

libVkLayer_khronos_validation.so 主要是调试用，  
`ImageProcessor::create(/*enableDebug=*/false, assetManager)` 传入 false 可以不需要   
可以在以下地址下载新版本    
<https://github.com/KhronosGroup/Vulkan-ValidationLayers>  

### RS、Vukan、RenderEffect、GLSL 效率对比

上文完整源码及对比示例地址   
<https://gitee.com/chenjim/android-blur/blob/blur/RenderScriptMigrationSample>   

他们之间效率对比结果如下  
```
processor = RenderScript Intrinsics, avg_ms = 5.772415893886967
processor = RenderScript Scripts, avg_ms = 27.450312792349727
processor = Vulkan, avg_ms = 9.245203770794829
processor = GLSL, avg_ms = 15.119627858006039
processor = ToolkitEffect, avg_ms = 8.924092
processor = RenderEffect, avg_ms = 4.4002173899999955
```
虽然 GLSL 看起来会差一些，主要是因为 openGL 纹理转 Bitmap 耗时较大。  
如果纯GL场景使用，跟 Vukan 和 RenderEffect 相差无几。  

---

以上就是Android 使用Vukan、RenderEffect、GLSL实现模糊的介绍，希望对你有所帮助。  
如果你在使用过程遇到问题，可以留言讨论。  
如果你觉得本文写的还不错，欢迎点赞收藏。  

---

**相关文章**  
[Android Bitmap 使用ScriptIntrinsicBlur、Toolkit 实现模糊](https://blog.csdn.net/CSqingchen/article/details/43817975)  
[Android Bitmap 使用Vukan、RenderEffect、GLSL实现模糊)](https://blog.csdn.net/CSqingchen/article/details/134656140)  


