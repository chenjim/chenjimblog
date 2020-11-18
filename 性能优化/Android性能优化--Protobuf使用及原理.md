

@[toc]

# Android 性能优化--Protobuf使用及原理
> 相关文章：[Gson和Fastjson的使用](https://blog.csdn.net/CSqingchen/article/details/48803267)


## 1. 什么是Protobuf？
- 官方开源地址 ：<https://github.com/protocolbuffers/protobuf>  
- 语法指南：<https://developers.google.com/protocol-buffers/docs/proto>  
- 本示例测试代码地址：<https://gitee.com/chenjim/ProtoBuf>
- google开源项目。序列化数据结构的方案，通常用于编写需要数据交换或者需要存储数据的程序。
这套方案包含一种用于描述数据结构的接口描述语言（Interface Description Language）和一个生成器，
用于生成描述该数据结构的不同编程语言的源代码。

### 1.1 优点
- 性能
	- 体积小，序列化后，数据大小可缩小3-10倍
	- 序列化速度快，比XML和JSON快20-100倍
	- 传输速度快，因为体积小，传输起来带宽和速度会有优化
- 使用
	- 使用简单，proto编译器自动进行序列化和反序列化
	- 维护成本低，多平台仅需维护一套对象协议文件(.proto)
	- 向后兼容性(扩展性)好，不必破坏旧数据格式就可以直接对数据结构进行更新
	- 加密性好，Http传输内容抓包只能看到字节

- 使用范围：跨平台、跨语言(支持[Java, Python, Objective-C, C+, Dart, Go, Ruby,
and C#等](https://developers.google.com/protocol-buffers/docs/tutorials)），可扩展性好


### 1.2 缺点
- 功能，不适合用于对基于文本的标记文档（如HTML）建模，因为文本不适合描述数据结构
- 通用性较差：json、xml已成为多种行业标准的编写工具，而Protobuf只是Google公司内部的工具
- 自解耦性差：以二进制数据流方式存储（不可读），需要通过.proto文件才能了解到数据结构

### 1.3 应用场景
- 传输数据量大 & 网络环境不稳定 的数据存储、RPC 数据交换 的需求场景
如 即时IM （QQ、微信）的需求场景
- 在传输数据量较大的需求场景下，Protobuf比XML、Json 更小、更快、使用 & 维护更简单！
- AS中就有使用protobuf，参见文件Android Studio\lib\protobuf-java-3.4.0.jar

## 2. 使用
### 2.1 安装 protoc
> protoc 是编译`*.proto`为相应语言的工具
####  2.1.1 ubuntu 安装 protoc
- 方法一  **apt-get**安装  
使用 `sudo apt-get install protoc` 安装
- 方法二 下载源码编译安装
1. 去[github下载*.tar.tz压缩包](https://github.com/protocolbuffers/protobuf/releases)，解压，
2. 命令行进入解压后目录，执行如下指令编译 `./configure && make `
3. 创建文件 /etc/ld.so.conf.d/libprotobuf.conf 写入内容：**/usr/local/lib**  
    输入命令 `sudo ldconfig`  
	注：protobuf的默认安装路径是/usr/local/lib，而/usr/local/lib不在Ubuntu体系默认的 LD_LIBRARY_PATH 里，后面install找不到该lib
4. `sudo make install `
5. `protoc --version` 验证是否安装成功
####  2.1.2 windows安装 protoc
1. 去[官方下载windows版本](https://developers.google.com/protocol-buffers/docs/downloads)解压
2. 把里面的protoc.exe所在目录,添加到环境变量


### 2.2 使用Android的Gradle插件编译

> protobuf-gradle-plugin官方介绍:<https://github.com/google/protobuf-gradle-plugin>  

- 修改项目根目录下build.gradle  
    dependencies下增加classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.8'
```
    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.0'
        classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.8'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
```

- 修改app目录下build.gradle  
    `apply plugin: 'com.android.application'`后加上`apply plugin: 'com.google.protobuf'`


- 然后加入protobuf配置（与android{}同级）
```
protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.0.0'
    }
    plugins {
        javalite {
            artifact = 'com.google.protobuf:protoc-gen-javalite:3.0.0'
        }
    }
    generateProtoTasks {
        all().each { task ->
            task.builtins {
                remove java
            }
            task.plugins {
                javalite { }
            }
        }
    }
}
```
- 最后dependencies下加入`implementation 'com.google.protobuf:protobuf-lite:3.+'`
- 安装**protobuf support**编辑插件,新建`test.proto`文件并编辑，
内容参考[test.proto](https://gitee.com/chenjim/ProtoBuf/blob/master/ProtobufAS/app/src/main/proto/test.proto)

- AS编译后生成文件在`app/build/generated/source/proto/debug/javalite`目录下，
包名同[test.proto](https://gitee.com/chenjim/ProtoBuf/blob/master/ProtobufAS/app/src/main/proto/test.proto)
中定义的java_package，
类文件名同[test.proto](https://gitee.com/chenjim/ProtoBuf/blob/master/ProtobufAS/app/src/main/proto/test.proto)
中定义的java_outer_classname

- 命令行编译：`protoc -I=./ --java_out=./ test.proto`，在当前目录生成同上的文件

- [protobuf-lite-3.0.0.jar下载地址](https://jcenter.bintray.com/com/google/protobuf/protobuf-lite/)

- 详细工程代码参考 <https://gitee.com/chenjim/ProtoBuf/tree/master/ProtobufAS> 

## 3. Protobuf序列化、反序列化的性能对比
参见源码[MainActivity.java](https://gitee.com/chenjim/ProtoBuf/blob/master/ProtobufAS/app/src/main/java/com/dn/protobuf/MainActivity.java)  中函数protoTest()

相对FastJson和Gson的对比参考代码同上，结果如下

```
E/MainActivity: protobuf 序列化耗时：4
E/MainActivity: protobuf 序列化数据大小：38
E/MainActivity: protobuf 反序列化耗时：2
E/JsonTest: FastJson 序列化耗时：13
E/JsonTest: FastJson 序列化数据大小：119
E/JsonTest: FastJson 反序列化耗时：16
E/JsonTest: Gson 序列化耗时：14
E/JsonTest: Gson 序列化数据大小：119
E/JsonTest: Gson 反序列化耗时：1
```
## 4. Protobuf 原理
- google官方原理介绍：<https://developers.google.com/protocol-buffers/docs/encoding>  
### 4.1 protobuf 数据结构
- 采用 TLV存储方式，即TAG-Length-Value(标识-长度-字段值)
- 不需要分隔符就能分隔开字段，减少了分隔符的使用;
- 各字段存储得非常紧凑，存储空间利用率非常高;
- 若字段没有被设置字段值，那么该字段在序列化时的数据中是完全不存在的，即不需要编码
<img src="https://pic.chenjim.com/20200323095304.png-blog" width = "600" />

### 4.2 protobuf 编码方式
对于不同数据类型 采用不同的 序列化方式（编码方式 & 数据存储方式）
<img src="https://pic.chenjim.com/20200323101928.png-blog" width = "600" />

### 4.3 TAG 编码过程
`Tag  = (field_number << 3) | wire_type `
wire_type 的值参考4.2，field_number就是字段标识号，详细[如下代码](https://gitee.com/chenjim/ProtoBuf/blob/master/ProtobufAS/app/src/main/proto/DNTest.proto)
```
    //wire_type=0, field_number=1
    //Tag  = (field_number << 3) | wire_type
    // TAG=8
    required int32 id1 = 1;
    
    //wire_type=0, field_number=2
    //Tag  = (field_number << 3) | wire_type
    // TAG=16
    required int32 id2 = 2;
```

### 4.4 Varint 编码过程
可参考源码[CodedOutputStream.java](https://github.com/protocolbuffers/protobuf/blob/master/java/core/src/main/java/com/google/protobuf/CodedOutputStream.java)
中1724行函数 writeUInt32NoTag

<img src="https://pic.chenjim.com/20200323102325.png-blog" width = "1000" />  

### 4.5 Varint 解码过程
<img src="https://pic.chenjim.com/20200323102625.png-blog" width = "1000" />


### 4.6 Zigzag 编码过程
<img src="https://pic.chenjim.com/20200323102832.png-blog" width = "700" />

### 4.7 总结建议
-  尽量使用多用 optional或 repeated修饰符  
因为若optional 或 repeated 字段没有被设置字段值，那么该字段在序列化时的数据中是完全不存在的，即不需要进行编码

- 字段标识号（Field_Number）尽量只使用 1-15，且不要跳动使用  
因为Tag里的Field_Number是需要占字节空间的。如果Field_Number>16时，Field_Number的编码就会占用2个字节，那么Tag在编码时也就会占用更多的字节；如果将字段标识号定义为连续递增的数值，将获得更好的编码和解码性能

- 若需要使用的字段值出现负数，请使用 sint32 / sint64，不要使用int32 / int64  
因为采用sint32 / sint64数据类型表示负数时，会先采用Zigzag编码再采用Varint编码，从而更加有效压缩数据

- 对于repeated字段，尽量增加packed=true修饰  
因为加了packed=true修饰repeated字段采用连续数据存储方式，即T - L - V - V -V方式


