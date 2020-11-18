
>本文主要讲述如何在jni中调用第三方共享so  
>源码地址：<https://gitee.com/chenjim/thirdPartyJniSo>
>本文地址：<http://blog.csdn.net/csqingchen/article/details/51548839>

### 如何使用

1. 生成第三方共享.so库  

    - 命令行切换到`thirdPartyJniSo/prebuild/`目录下  
    - 执行`ndk-build`在libs目录生成Android平台各种CPU指令集的库

2. Eclise使用示例
   
    - 命令行切换到`hirdPartyJniSo\thirdPartyJniSoEclipse`,
    - 执行`ndk-build`在libs中生成HelloJni.java需要的.so库  
    - Eclipse打开导入thirdPartyJniSoEclipse，待验证。
    
3. Android Studio使用示例

    - 复制`thirdPartyJniSo\prebuild\libs` 中目录到 `thirdPartyJniSoAS/app/src/main/jniLibs/`
    - 复制头文件`thirdPartyJniSo\prebuild\jni\add_test.h`到`thirdPartyJniSo\thirdPartyJniSoAS\app\src\main\cpp\include\add_test.h`
    - AS打开`thirdPartyJniSo\thirdPartyJniSoAS即可build

    

### 注意问题

1. [gcc 生成的共享动态库](http://blog.csdn.net/csqingchen/article/details/51546784)，android无法使用，必须用ndk编译生成的.so，否则提示".so: File format not recognized"。  
2. 项目中Application.mk 中APP_ABI的配置必须相同，否则提示".so: File format not recognized"。  
3. 最终调用的libhello-jni.so也是动态共享库，为啥不直接用`thirdPartyJniSo/prebuild/libs`中的`libadd_test.so`？是因为HelloJni.java只能通过native桥接调用so文件。  

**原创文章，转载请注明出处、原文链接！  
邮件<me@h89.cn> 主页[https://chenjim.com](https://h89.cn)**

参考文章：

[google ndk 示例代码hello-libs](https://github.com/android/ndk-samples/tree/master/hello-libs)
[google Android abis 介绍](https://developer.android.com/ndk/guides/abis.html)
[CSDN Android.mk库编译](http://blog.csdn.net/educast/article/details/12773127)  
[gcc/g++与makefile](https://www.jianshu.com/p/9ce282dc3966/)


