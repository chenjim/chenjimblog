
[TOC]

> 本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/51896769>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  

## 前言
- 良好的代码格式方便阅读，利于多人维护，避免格式化代码后差异太多。
- android studio默认配置格式化后，基本满足我们的格式要求。
- 本位主要介绍如何安装，配置checkstyle。
- 部分内容参考自[官方文档](https://checkstyle.sourceforge.io/)，目前流行的两种范约束xml分别是[sun_checks.xml](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/sun_checks.xml ) 和[google_checks.xml](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml ) ，本文使用的[chenjim_checks.xml](https://gitee.com/chenjim/chenjimblog/blob/master/data/checkstyle/chenjim_checks.xml ) 依赖 [checkstyle-6.8.1-all.jar](https://github.com/checkstyle/checkstyle/releases/tag/checkstyle-6.8.1 ) , 更高版本可能需要适当修改。  

## android studio（intellij）中安装使用
1. 要求STUDIO_JDK为jdk8以上,否则安装后无法正常使用，  
   可以在环境变量中配置`STUDIO_JDK=../jdk`为studio的jdk版本
2. 在线安装，File-->setting-->Plugins-->Marketplace,搜索checkstyle，然后安装，如下图  
   ![](https://pic.chenjim.com/201906031907019.png-blog)
3. 离线安装，File-->setting-->Plugins-->install plugin from disk，如下图，选择 [官方的文件安装包](https://plugins.jetbrains.com/plugin/1065-checkstyle-idea/versions)  
![](https://pic.chenjim.com/2019060318595726.png-blog)
4. 导入配置文件： 打开studio，File-->Settings-->OtherSettings-->CheckStyle，  
   点“+”，选中“Use a local checkstyle file”，点击`Browse`，  
   导入[配置文件chenjim_checks.xml](https://gitee.com/chenjim/chenjimblog/blob/master/data/checkstyle/chenjim_checks.xml )，然后**只勾选**刚导入的，  
   点击 `OK`，退出 `Setting`
5. 如何使用：打开studio，打开java代码，右键，点击`Check current file` 选项，  
   检测当前代码是否有checkstyle异常，下方checkStyle栏目会显示检测结果.

本文地址 <http://blog.csdn.net/csqingchen/article/details/51896769>

## eclipse中安装使用
1. 菜单help-->install new software，在`work with`后面输入栏，输入在线安装地址 http://eclipse-cs.sourceforge.net/update，安装
2. 菜单Windows-->Preference，点击`checkstyle`，点击`New`，在`Check Configuration Properties`界面，`type`选择`External Configuration File`， 点击`Browse`，导入[chenjim_checks.xml](https://gitee.com/chenjim/chenjimblog/blob/master/data/checkstyle/chenjim_checks.xml) ，配置完成点击`OK`.
3. 选中刚新加的配置，点击右侧`Set as Default`
4. 打开一份java代码，右键，就有checkstyle选项

## 命令行中使用
>使用较繁琐，版本不一样还可能会出现其他异常，不推荐使用

1. 从[checkstyle releases @github](https://github.com/checkstyle/checkstyle/releases) 下载`checkstyle-**-all.jar`到自己的电脑  
2. 使用命令形式如下   
`java -jar checkstyle-**-all.jar的绝对路径 -c chenjim_checks.xml的绝对路径 要检测文件的路径`  
例如  
`java -jar /home/chenjim/bin/checkstyle/checkstyle-6.8.1-all.jar -c /home/chenjim/bin/checkstyle/chenjim_checks.xml ./MainActivity.java`   
3. 命令太长了，太难记，太麻烦了，可以写一个脚本文件checkstyle/checkstyle.bat，里面内容如下  
`java -jar /home/chenjim/bin/checkstyle/checkstyle-6.8.1-all.jar -c /home/chenjim/bin/checkstyle/chenjim_checks.xml $1`  
将文件加到环境变量，加上可执行权限，然后 `checkstyle ./MainActivity.java` 就行了
