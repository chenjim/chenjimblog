
> 本文首发地址：<https://h89.cn/archives/13.html>  
> 最新更新地址：<https://gitee.com/chenjim/chenjimblog>

在 ITelephony.aidl  [ 8.0 源码](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/oreo-m8-release/telephony/java/com/android/internal/telephony/ITelephony.aidl) 、[9.0 源码 ](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/pie-release/telephony/java/com/android/internal/telephony/ITelephony.aidl) 中存在 `endCall()` 接口  
在 [10  源码](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android10-release/telephony/java/com/android/internal/telephony/ITelephony.aidl) 中，已经没有  `endCall()`  接口  
在 Android 10 之前可以通过如下方式 挂断 电话
```java
//详细 参见   https://www.jianshu.com/p/a5662fad84b5  
public void endCall() {
    try {
        //1.通过类加载器加载相应类的class文件
        //Class<?> forName = Class.forName("android.os.ServiceManager");
        Class<?> loadClass = getClassLoader().loadClass("android.os.ServiceManager");
        //2.获取类中相应的方法
        //name : 方法名
        //parameterTypes : 参数类型
        Method method = loadClass.getDeclaredMethod("getService", String.class);
        //3.执行方法,获取返回值
        //receiver : 类的实例
        //args : 具体的参数
        IBinder invoke = (IBinder) method.invoke(null, Context.TELEPHONY_SERVICE);
        //aidl
        ITelephony iTelephony = ITelephony.Stub.asInterface(invoke);
        //挂断电话
        iTelephony.endCall();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
----
那 Android 10 如何挂断电话呢？
在 TelephonyManager.java  [9.0 源码 ](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/pie-release/telephony/java/android/telephony/TelephonyManager.java) 我们可以看到如下，也就是前面实现挂断电话的 framework 原理  
```java
@SystemApi
@RequiresPermission(android.Manifest.permission.CALL_PHONE)
public boolean endCall() {
    try {
        ITelephony telephony = getITelephony();
        if (telephony != null)
            return telephony.endCall();
    } catch (RemoteException e) {
        Log.e(TAG, "Error calling ITelephony#endCall", e);
    }
    return false;
}
...
private ITelephony getITelephony() {
    return ITelephony.Stub.asInterface(ServiceManager.getService(Context.TELEPHONY_SERVICE));
}

```

在 TelephonyManager.java [10.0 源码](https://android.googlesource.com/platform/frameworks/base.git/+/refs/heads/android10-release/telephony/java/android/telephony/TelephonyManager.java)  中我们可以看到如下 
```java
    /**
     * @removed Use {@link android.telecom.TelecomManager#endCall()} instead.
     * @hide
     * @removed
     */
    @Deprecated
    @SystemApi
    @RequiresPermission(android.Manifest.permission.CALL_PHONE)
    public boolean endCall() {
        return false;
    }
```
是不是可以使用 `android.telecom.TelecomManager#endCall()` 呢 ？

看下 TelecomManager.java 源码 [9.0](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/pie-release/telecomm/java/android/telecom/TelecomManager.java) 、 [10.0](https://android.googlesource.com/platform/frameworks/base.git/+/refs/heads/android10-release/telecomm/java/android/telecom/TelecomManager.java)，可以看到如下   
```java
    @RequiresPermission(Manifest.permission.ANSWER_PHONE_CALLS)
    @Deprecated
    public boolean endCall() {
        try {
            if (isServiceConnected()) {
                return getTelecomService().endCall(mContext.getPackageName());
            }
        } catch (RemoteException e) {
            Log.e(TAG, "Error calling ITelecomService#endCall", e);
        }
        return false;
    }
```
所以问题不大了，最终代码如下  
```java
AndroidManifest.xml 需 添加以下权限
<uses-permission android:name="android.permission.ANSWER_PHONE_CALLS" />
<uses-permission android:name="android.permission.CALL_PHONE" />

//保持 前面 ITelephony.aidl 包名相对路径复制到 应用代码  

//注意动态申请权限(Permission.ANSWER_PHONE_CALLS)
private fun endCallAction() {
    try {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
         	val tcm = context.getSystemService(Context.TELECOM_SERVICE) as TelecomManager
            tcm.endCall()
        } else {
            val loadClass: Class<*> =
                javaClass.classLoader.loadClass("android.os.ServiceManager")
            val method: Method = loadClass.getDeclaredMethod("getService", String::class.java)
            //这里也可以直接用 context.getSystemService(Context.TELEPHONY_SERVICE ) as TelephonyManager
            //注意 TELECOM* 和 TELEPHONY* 区别 
            val invoke: IBinder = method.invoke(null, Context.TELEPHONY_SERVICE) as IBinder
            val iTelephony: ITelephony = ITelephony.Stub.asInterface(invoke)
            iTelephony.endCall()
        }
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```

----

这就完了，还没有，我们再看下 TelecomManager.java 中 endcall 注释   
```java 
     * @deprecated Companion apps for wearable devices should use the {@link InCallService} API
     * instead.  Apps performing call screening should use the {@link CallScreeningService} API instead.
     */
    @RequiresPermission(Manifest.permission.ANSWER_PHONE_CALLS)
    @Deprecated
    public boolean endCall() {   ...   }
```

 @Deprecated    @Deprecated    @Deprecated   
   
 也就是将来也可能会移除，那时怎么做呢？
 上面注释中看到可穿戴设备用 `InCallService`，   其它APP用 `CallScreeningService`    
 我们看下 InCallService.java [10.0 源码  ](https://android.googlesource.com/platform/frameworks/base.git/+/refs/heads/android10-release/telecomm/java/android/telecom/InCallService.java)  和 CallScreeningService.java [10.0 源码 ](https://android.googlesource.com/platform/frameworks/base.git/+/refs/heads/android10-release/telecomm/java/android/telecom/CallScreeningService.java)    

 哇，类注释都非常详细，需要时参考下就好，本文暂时结束。





