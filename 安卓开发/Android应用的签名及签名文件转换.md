[TOC]
>**keytool**、**jarsigner** 均是jdk提供的工具，[JDK下载链接](http://www.oracle.com/technetwork/java/javase/downloads/index.html)  
>涉及的文件可以参考示例 [AndroidSignKeyConvert](https://gitee.com/chenjim/AndroidSignKeyConvert)  
>Android 签名机制 v1、v2、v3 参阅 <https://source.android.com/security/apksigning/v2>  
>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/78228933>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>  

本文主要介绍安卓应用签名相关内容，包括签名文件的生成，如何对APK签名，如何查看签名结果，以及常见签名之间的相互转换。

### 生成APK签名文件

#### 使用keytool命令生成.jks
`keytool -genkeypair -keystore ./genkey.jks -storepass 123123 -alias business -keypass 123456 -dname "cn=chenjim, ou=h89, o=cn,L="sh",ST="sh", c=cn" -validity 120000 -keyalg RSA`   
将会生成签名文件`./genkey.jks `,有效期`1200`个月,别名`business `,密钥库的密码`123123`，`business `的密码`123456`  
后面的RSA是签名加密类型，建议用这个，DSA在有些平台会不识别  
```
-keyalg
    "DSA" (when using -genkeypair)
    "DES" (when using -genseckey)

-keysize
    2048 (when using -genkeypair and -keyalg is "RSA")
    1024 (when using -genkeypair and -keyalg is "DSA")
    256 (when using -genkeypair and -keyalg is "EC")
    56 (when using -genseckey and -keyalg is "DES")
    168 (when using -genseckey and -keyalg is "DESede")
```
可将`./genkey.jks`直接换`./genkey.keystore`生成.keystore签名文件  
参数含义可参考 <https://docs.oracle.com/javase/9/tools/keytool.htm>  


---

#### 使用 android studio 的生成如下  
菜单 Build-->Generate Signed APK-->Create new

---

### 对apk签名的几种方式
#### 用指定的keystore(jks) 对 apk 签名 
1. 用jarsigner对apk签名，这个支持V1签名即jar签名  
    ```
    jarsigner -keystore ./genkey.jks -signedjar ./signed.apk ./unsigned.apk business
    输入密钥库的密码短语: //123123
    输入business的密钥口令: //123456
    jar 已签名。
    ```
    将`unsigned.apk`用 `./genkey.jks` 的`business` 密匙签名为`signed.apk`  
    [示例](https://gitee.com/chenjim/AndroidSignKeyConvert) 中的 `unsigned.apk` 是一个已经DSA签名的 apk，这里给重新签名    
2. 使用 apksigner 对 apk 签名  
    `apksigner.bat sign --ks ./genkey.jks signed.apk`  
    - `apksigner`  来自 `..\android-sdk\build-tools\31.0.0\apksigner`
    - `apksigner sign --help` 命令的帮助说明  

---

#### 使用私钥和证书(platform.x509.pem和platform.pk8)对 apk 签名
此种情况多见Android系统开发中，对系统应用签名，有如下种命令
1. `apksigner sign --key genkey.pk8 --cert genkey.x509.pem --in unsigned.apk --out signed.apk`  
这个是 android sdk 提供的签名方式，参考 [apksigner](https://developer.android.com/studio/command-line/apksigner)，也有对V1、V2支持的讲叙  


1. `java -jar signapk.jar  platform.x509.pem platform.pk8 unsigned.apk signed.apk`
   - **signapk.jar** 可以编译Android源码(`mmm build/tools/signapk/`) 得到，源码中位置 `prebuilts/sdk/tools/lib/signapk.jar`也可能有
   - **platform.x509.pem和platform.pk8**在源码目录 `build/target/product/security/` 中


---

#### 使用集成IDE对APK签名
多数IDE(IntellJ/AS)是在菜单**build**-->**Generate Signed APK**中  
Android Studio 可以自动签名，需要在build.gradle中添加如下配置  
```groovy
signingConfigs {
    config {
        storeFile file('../genkey.jks')
        storePassword '123123'
        keyAlias "business"
        keyPassword '123456'
    }
}

buildTypes {
    release {
        minifyEnabled false
        signingConfig signingConfigs.config
    }
}
```

本文地址 <http://blog.csdn.net/CSqingchen/article/details/78228933>


---

### 查看签名信息

#### 查看签名文件(jks)的签名信息
`keytool -list -v -keystore genkey.jks`  
需要用到之前生成签名文件的密钥库的密码`123123`
```bash
发布者: CN=Chenjim, OU=chenjim, O=chenjim, L=Shanghai, ST=Minhang, C=CN
序列号: 654badf0
生效时间: Thu Mar 26 14:32:11 CST 2020, 失效时间: Wed Oct 13 14:32:11 CST 2348
证书指纹:
         SHA1: CC:A0:52:3A:4A:6F:0A:77:3A:68:2C:A0:18:52:1D:A7:36:EB:B5:16
         SHA256: A3:F7:B3:7E:3F:C0:E6:A8:FF:74:2C:4E:FA:FC:AC:66:E4:38:B3:02:2C:94:7E:07:AC:63:B2:0F:30:7F:09:51
签名算法名称: SHA1withDSA (弱)
主体公共密钥算法: 1024 位 DSA 密钥 (弱)
```

---

#### 用apksigner查看apk签名V1 V2 V3 支持情况  
   命令 `apksigner verify -v signed.apk`  
   结果类似如下  
   ```
   Verified using v1 scheme (JAR signing): true
   Verified using v2 scheme (APK Signature Scheme v2): true
   Verified using v3 scheme (APK Signature Scheme v3): false
   Number of signers: 1
   ```
   《Android v1、v2、v3签名详解》可参考 <https://source.android.com/security/apksigning/v2>

--- 

#### 用 keytool 查看 apk 签名
   1. 将签名后的`signed.apk`解压可得到签名文件 `META-INF\BUSINESS.RSA`，可能后缀为 `*.DSA`，由前面生成的 jks 加密类型决定   
   `keytool -printcert -file ./BUSINESS.DSA` 显示结果   
   2. 直接使用如下命令  
   `keytool -printcert -jarfile signed.apk`  

---

#### 在项目中用gradle查看签名信息

   执行命令`.\gradlew.bat :app:signingReport`，得到如下签名结果  
   ```bash
   Variant: release
   Config: config
   Store: ....AndroidSignKeyConvert\genkey.jks
   Alias: business
   MD5: 9F:E0:78:DD:BC:2A:C2:FB:90:8D:67:D1:F7:FE:29:BF
   SHA1: CC:A0:52:3A:4A:6F:0A:77:3A:68:2C:A0:18:52:1D:A7:36:EB:B5:16
   SHA-256: A3:F7:B3:7E:3F:C0:E6:A8:FF:74:2C:4E:FA:FC:AC:66:E4:38:B3:02:2C:94:7E:07:AC:63:B2:0F:30:7F:09:51
   Valid until: 2348年10月13日 星期三
   ```
---

#### 应用代码中获取APK签名信息
>参考自 <https://blog.csdn.net/anydrew/article/details/51227517>  
>普通APP只能获取自己的签名信息，无法获取其它应用的签名  
核心代码整理如下，参见 [测试代码 MainActivity.java](https://gitee.com/chenjim/AndroidSignKeyConvert/blob/master/app/src/main/java/com/chenjim/andsignkeyconvert/MainActivity.java)  
```JAVA
public String getAppSign() {
    String result = null;
    PackageManager packageManager = getPackageManager();
    try {
        PackageInfo packageInfo = packageManager.getPackageInfo(
                   getPackageName(), PackageManager.GET_SIGNATURES);
        if (packageInfo == null) {
            return null;
        }
        Signature[] signatures = packageInfo.signatures;
        if (signatures == null || signatures.length < 1) {
            return null;
        }
        Signature signature = signatures[0];
        MessageDigest messageDigest = MessageDigest.getInstance("MD5");
        //messageDigest = MessageDigest.getInstance("SHA-1");
        //取MD5的值并转String
        byte[] md5Bytes = messageDigest.digest(signature.toByteArray());
        //对生成的16字节数组进行补零操作
        StringBuilder md5 = new StringBuilder(md5Bytes.length * 2);
        for (byte b : md5Bytes) {
            if ((b & 0xFF) < 0x10) {
                md5.append("0");
            }
            md5.append(Integer.toHexString(b & 0xFF));
        }
        result = md5.toString();
    } catch (Exception e) {
        e.printStackTrace();
    }
    Log.d("getAppSign", "getAppSign result:" + result);
    return result;
}
```

---

### 签名文件转换

#### .jks 转 .p12
```bash
keytool -importkeystore -srckeystore ./genkey.jks -srcstoretype JKS -deststoretype PKCS12 -destkeystore genkey.p12
输入目标密钥库口令://123456
再次输入新口令://123456
输入源密钥库口令://123123
输入 <business> 的密钥口令 //123456
已成功导入别名 business 的条目。
已完成导入命令: 1 个条目成功导入, 0 个条目失败或取消
```

---

#### .p12 转 .keystore
```bash
keytool -v -importkeystore -srckeystore ./genkey.p12 -srcstoretype PKCS12 -destkeystore ./genkey.keystore -deststoretype JKS
输入目标密钥库口令://123123
再次输入新口令://123123
输入源密钥库口令://123456
已成功导入别名 business 的条目。
已完成导入命令: 1 个条目成功导入, 0 个条目失败或取消
[正在存储./genkey.keystore]
```
可以用以下命令效验.jks和.keystore签名数据，密码都是123123
```bash
keytool -list -v -keystore genkey.jks
keytool -list -v -keystore genkey.keystore
```

---

#### .keystore转 .x509.pem和.pk8
> 需要用到openssl，windows可以在Git安装目录找到，如'C:\Program Files\Git\mingw64\bin'，linux中没有的自行安装

1. 将PKCS12格式的key dump为可直接阅读的文本
```bash
openssl pkcs12 -in genkey.p12 -nodes -out genkey.rsa.pem
Enter Import Password: //参考如上5.1 p12的密码123456
```
2. 提取genkey.rsa.pem为genkey_private.rsa.pem和genkey.x509.pem
```
用文本编辑器打开 genkey.rsa.pem，将从
-----BEGIN PRIVATE KEY-----
 到
-----END PRIVATE KEY-----
这一段（包含这两个tag）的文本复制出来，新建为文件 genkey_private.rsa.pem
将从
-----BEGIN CERTIFICATE-----
到
-----END CERTIFICATE-----
这一段（包含这两个tag）的文本复制出来，新建为文件 genkey.x509.pem （签名时用到的公钥）
```

3. 生成pk8格式的私钥  
   `openssl pkcs8 -topk8 -outform DER -in genkey_private.rsa.pem -inform PEM -out genkey.pk8 -nocrypt`  

4. 至此步骤3.2中的.x509.pem和.pk8 已经生成，可以用如下命令签名  
   `apksigner sign --key genkey.pk8 --cert genkey.x509.pem --in unsigned.apk --out signed.apk`  

---

#### pem 转 jks 

参见 [ssl_convert_pem_to_jks](https://docs.oracle.com/cd/E35976_01/server.740/es_admin/src/tadm_ssl_convert_pem_to_jks.html)
提取如下（未验证）      
```
//pem 转 PKCS12
openssl pkcs12 -export -out eneCert.pkcs12 -in eneCert.pem
//keytool -genkey -keyalg RSA -alias endeca -keystore keystore.ks
keytool -genkey -keyalg RSA -alias endeca -keystore keystore.ks
//Import your private key into the empty JKS
keytool -v -importkeystore -srckeystore eneCert.pkcs12 -srcstoretype PKCS12 -destkeystore keystore.ks -deststoretype JKS
```

---


**原创文章，转载请注明出处、原文链接！<me@h89.cn>   我的主页<https://chenjim.com>**

参考文章  
[Docs About **keytool** By Oracle](http://docs.oracle.com/javase/8/docs/technotes/tools/windows/keytool.html)  
[Android 根据包名，获取应用程序的签名](http://blog.csdn.net/anydrew/article/details/51227517)  
[java keytool证书工具使用小结](http://www.micmiu.com/lang/java/keytool-start-guide/)  
[Android将jks签名文件转为keystore](https://blog.csdn.net/u014549283/article/details/81190034)  


