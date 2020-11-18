
[TOC]

> 本文首发地址 <https://h89.cn/archives/190.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  


在应用开发过程中，我们经常需要动态计算状态栏高度，本文介绍几种方法。


## 方法一 直接使用24dp


在[Android 9.0 frameworks/base/core/res/res/](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/pie-release-2/core/res/res/values/dimens.xml)中有如下    
```
    <!-- Height of the status bar -->
    <dimen name="status_bar_height">@dimen/status_bar_height_portrait</dimen>
    <!-- Height of the status bar in portrait -->
    <dimen name="status_bar_height_portrait">24dp</dimen>
    <!-- Height of the status bar in landscape -->
    <dimen name="status_bar_height_landscape">@dimen/status_bar_height_portrait</dimen>
```
可以看到`status_bar_height`只有一个定值24dp，因此可以直接使用

Android 9.0的frameworks/base/core/res/res目录源码：<https://android.googlesource.com/platform/frameworks/base/+archive/refs/heads/pie-release-2/core/res/res.tar.gz>  
同理 `navigation_bar_height` 可以直接用48dp

**要想做到完美兼容，不建议直接用固定值，部分定制设备可能有其他值。**

## 方法二 getDimensionPixelSize
获取系统中"status_bar_height"的值，方法如下
```
public static int getStatusBarHeight(Context context) {
    Resources resources = context.getResources();
    int resourceId = resources.getIdentifier("status_bar_height", "dimen", "android");
    int height = resources.getDimensionPixelSize(resourceId);
    return height;
}
```

## 方法三 利用反射
```
public static int getStatusHeight(Activity activity) {
    int statusHeight = 0;
    try {
        Class<?> localClass = Class.forName("com.android.internal.R$dimen");
        Object localObject = localClass.newInstance();
        int h = Integer.parseInt(localClass.getField("status_bar_height").get(localObject).toString());
        statusHeight = activity.getResources().getDimensionPixelSize(h);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return statusHeight;
}
```
本文到此结束，如果你在使用遇到问题，欢迎留言讨论。
