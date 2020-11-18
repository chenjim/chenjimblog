

@[toc]

>**keytool**、**jarsigner** 均是jdk提供的工具，[JDK下载](http://www.oracle.com/technetwork/java/javase/downloads/index.html)  
>涉及的文件可以参考示例 [AndroidSignKeyConvert](https://gitee.com/chenjim/AndroidSignKeyConvert)
>Android 签名机制 v1、v2、v3 参阅 <https://www.jianshu.com/p/9ca1d6f3f083>
### 1. 生成APK签名文件
#### 1.1 使用keytool命令生成.jks
`keytool -genkeypair -alias business -keypass 123456 -keystore ./genkey.jks -storepass 123123 -validity 120000 -keysize 1024`   
将会生成签名文件`./genkey.jks `,有效期`1200`个月,别名`business `,密钥库的密码`123123`，`business `的密码`123456`  
可将`./genkey.jks`直接换`./genkey.keystore`生成.keystore签名文件
#### 1.2. 使用IDE生成，IntellJ的生成如下
菜单 Build-->Generate Signed APK-->Create new

### 2. 查看签名文件的签名信息
`keytool -list -v -keystore genkey.jks`  
需要用到之前生成签名文件的密钥库的密码`123123`

### 3. 对apk签名的几种方式
#### 3.1 用指定的keystore 签名apk
```
jarsigner -keystore ./genkey.jks -signedjar ./signed.apk ./unsigned.apk business
输入密钥库的密码短语: //123123
输入business的密钥口令: //123456
jar 已签名。
```
将`unsigned.apk`用`./genkey.jks`的`business`密匙签名为`signed.apk`
需要用到之前生成签名文件的两个密码

#### 3.2 使用platform.x509.pem和platform.pk8对apk签名
>此种情况多见Android系统开发中，对系统应用签名  
>有如下新旧两种命令

1. `java -jar apksigner.jar sign --key genkey.pk8 --cert genkey.x509.pem --in unsigned.apk --out signed.apk`
- 这个是新的签名方式，参考[apksigner](https://developer.android.com/studio/command-line/apksigner)
- apksigner.jar 来自..\sdk\build-tools\29.0.0\lib\
- `java -jar apksigner.jar sign --help`命令的帮助说明

2. `java -jar signapk.jar  platform.x509.pem platform.pk8 unsigned.apk signed.apk`
-  **signapk.jar** 可以编译Android源码(`mmm build/tools/signapk/`) 得到，源码中位置`prebuilts/sdk/tools/lib/signapk.jar`也可能有
- **platform.x509.pem和platform.pk8**在源码目录`build/target/product/security/`中

#### 3.3 使用集成IDE对APK签名
多数IDE(IntellJ/AS)是在菜单**build**-->**Generate Signed APK**中  
  
Android Studio 可以自动签名，需要在build.gradle中添加如下配置
```
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

### 4. 用keytool查看应用(APK)的签名
- 将签名后的`signed.apk`解压缩到目录signed，得到签名文件`.\signed\META-INF\BUSINESS.DSA`，有些是`META-INF\CERT.RSA`
- 使用keytool可以查看  `keytool -printcert -file ./BUSINESS.DSA`

本文地址<http://blog.csdn.net/CSqingchen/article/details/78228933>

### 5. 签名文件转换之 .jks转 .keystone

#### 5.1 .jks 转 .p12
```
keytool -importkeystore -srckeystore ./genkey.jks -srcstoretype JKS -deststoretype PKCS12 -destkeystore genkey.p12
输入目标密钥库口令://123456
再次输入新口令://123456
输入源密钥库口令://123123
输入 <business> 的密钥口令 //123456
已成功导入别名 business 的条目。
已完成导入命令: 1 个条目成功导入, 0 个条目失败或取消
```
#### 5.2 .p12 转 .keystore
```
keytool -v -importkeystore -srckeystore ./genkey.p12 -srcstoretype PKCS12 -destkeystore ./genkey.keystore -deststoretype JKS
输入目标密钥库口令://123123
再次输入新口令://123123
输入源密钥库口令://123456
已成功导入别名 business 的条目。
已完成导入命令: 1 个条目成功导入, 0 个条目失败或取消
[正在存储./genkey.keystore]
```
可以用以下命令效验.jks和.keystore签名数据，密码都是123123
```
keytool -list -v -keystore genkey.jks
keytool -list -v -keystore genkey.keystore
```
### 6. 签名文件转换之 .keystore转.x509.pem和.pk8
> 需要用到openssl，windows可以在Git安装目录找到，如'C:\Program Files\Git\mingw64\bin'，linux中没有的自信安装

1. 将PKCS12格式的key dump为可直接阅读的文本
```
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
`java -jar apksigner.jar sign --key genkey.pk8 --cert genkey.x509.pem --in unsigned.apk --out signed.apk`

### 7. 应用中获取签名信息
>参考自<https://blog.csdn.net/anydrew/article/details/51227517>
>普通APP只能获取自己的签名信息，无法获取其它应用的
核心代码整理如下，参见[测试代码 MainActivity.java](https://gitee.com/chenjim/AndroidSignKeyConvert/blob/master/app/src/main/java/com/chenjim/andsignkeyconvert/MainActivity.java)
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

**原创文章，转载请注明出处、原文链接！<me@h89.cn>   我的主页<https://chenjim.com>**

参考文章  
[Docs About **keytool** By Oracle](http://docs.oracle.com/javase/8/docs/technotes/tools/windows/keytool.html)  
[Android 根据包名，获取应用程序的签名](http://blog.csdn.net/anydrew/article/details/51227517)  
[java keytool证书工具使用小结](http://www.micmiu.com/lang/java/keytool-start-guide/)  
[Android将jks签名文件转为keystore](https://blog.csdn.net/u014549283/article/details/81190034)  

