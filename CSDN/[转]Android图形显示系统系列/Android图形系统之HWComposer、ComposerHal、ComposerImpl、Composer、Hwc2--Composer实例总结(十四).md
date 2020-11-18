**原文** [Android图形系统之HWComposer、ComposerHal、ComposerImpl、Composer、Hwc2--Composer实例总结(十四)](https://unbroken.blog.csdn.net/article/details/134142730)  

**简介：** **CSDN博客专家，专注Android/Linux系统，分享多mic语音方案、音视频、编解码等技术，与大家一起成长！**

**优质专栏：**[**Audio工程师进阶系列**](https://blog.csdn.net/u010164190/category_9599435.html)【**原创干货持续更新中……**
】🚀

**人生格言：** **人生从来没有捷径，只有行动才是治疗恐惧和懒惰的唯一良药.**

更多原创,欢迎关注：Android系统攻城狮

![欢迎关注Android系统攻城狮](https://img-
blog.csdnimg.cn/20190106163945739.jpg#pic_center)

### 1.前言

>
> 本篇目的：Android图形系统中，HWC特别的复杂，特别是HWComposer、ComposerImpl、Composer、Hwc2::Composer之间的关系，有种剪不断理还乱的感觉，通过几个演化例子，看清楚它本来的面目。

### 2.HWComposer、ComposerImpl、Composer、Hwc2::Composer定义与实现

#### 1.HWComposer实现

frameworks/native/services/surfaceflinger/DisplayHardware/HWComposer.h

##### <1>.android::HWComposer定义

    
    
    namespace Hwc2 {
       
    class Composer;
    }
    
    namespace android {
       
    class HWComposer {
       
    	virtual ~HWComposer();
        virtual void setCallback(HWC2::ComposerCallback*) = 0;
        ....
    };
    }
    

##### <2>.impl::HWComposer定义(继承自android::HWComposer)

    
    
    namespace impl {
       
    
    class HWComposer final : public android::HWComposer {
       
    public:
        explicit HWComposer(std::unique_ptr<Hwc2::Composer> composer);
        explicit HWComposer(const std::string& composerServiceName);
    
        ~HWComposer() override;
    };
    }
    

##### <3>.impl::HWComposer实现

frameworks/native/services/surfaceflinger/DisplayHardware/HWComposer.cpp

    
    
    namespace impl {
       
    	HWComposer::HWComposer(std::unique_ptr<Hwc2::Composer> composer)
          : mComposer(std::move(composer)),
            mMaxVirtualDisplayDimension(static_cast<size_t>(sysprop::max_virtual_display_dimension(0))),
            mUpdateDeviceProductInfoOnHotplugReconnect(
                    sysprop::update_device_product_info_on_hotplug_reconnect(false)) {
       }
    
    HWComposer::HWComposer(const std::string& composerServiceName)
          : HWComposer(std::make_unique<Hwc2::impl::Composer>(composerServiceName)) {
       }
    
    HWComposer::~HWComposer() {
       
        mDisplayData.clear();
    }
    }
    

>
> HWComposer构造函数通过它的委托构造函数，通过std::make_uniqueHwc2::impl::Composer(composerServiceName)实例画，那么Hwc2::impl::Composer的实现在哪呢？

#### 2.ComposerImpl实现（Hwc2::Composer是它的别名）

hardware/interfaces/graphics/composer/2.1/utils/hal/include/composer-
hal/2.1/Composer.h

##### <1>.Composer定义，它是ComposerImpl的别名

    
    
    using Composer = detail::ComposerImpl<IComposer, ComposerHal>;
    template <typename Interface, typename Hal>
    class ComposerImpl : public Interface {
       
       public:
        static std::unique_ptr<ComposerImpl> create(std::unique_ptr<Hal> hal) {
       
            

