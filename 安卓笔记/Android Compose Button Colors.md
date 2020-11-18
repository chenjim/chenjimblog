
# Android Compose Button defaultButtonColors

> 本文最新更新地址 <https://gitee.com/chenjim/chenjimblog>

### 发现问题
最近看 Android Compose 相关资料发现如下代码  
```kotlin
    colors = defaultButtonColors(
        backgroundColor = if (count > 5) Color.Green else Color.White
    )
```

![](https://pic.chenjim.com/202306301351427.png-blog)  

原文地址 <https://developer.android.com/jetpack/compose/preview?hl=zh-cn>  

编译会出现异常 `Unresolved reference: defaultButtonColors`  

---

### 解决问题 
1. 以上是中文页面，对应的 [英文页面](https://developer.android.com/jetpack/compose/preview) ，当前(20230701) 已经没有相应的说明  
   新版 compose preview 介绍参考 <https://developer.android.com/jetpack/compose/tooling/previews> 

2. 在新版本中，本文使用的是 `implementation 'androidx.compose.material3:material3:1.1.1'`  
    已经没有 `ButtonConstants.defaultButtonColors` 和 `backgroundColor`    
    可以使用如下代码替换  
    ```kotlin
        colors = ButtonDefaults.buttonColors(
            containerColor = if (count > 5) Color.Green else Color.Gray
        )
    ```
3. 在 [android-compose-codelabs](https://github.com/googlecodelabs/android-compose-codelabs.git) 示例中，也均使用的是 `ButtonDefaults.buttonColors`  

---

参考文档  
<https://stackoverflow.com/questions/64376333>  
<https://developer.android.com/jetpack/compose/documentation>  
