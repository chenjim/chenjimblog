
@[toc]
>1. 良好的代码格式方便阅读，利于多人维护，避免格式化代码后差异太多
>2. android studio默认配置格式化后，基本满足我们的格式要求。
>3. 下面是如何安装，配置checkstyle。
>4. 部分内容参考自[官方文档](http://checkstyle.sourceforge.net/ )，目前主要是有两种范约束xml文件，分别是[sun_checks.xml](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/sun_checks.xml)和[google_checks.xml](hhttps://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml)，本人使用的[简化配置文件](https://gitee.com/chenjim/checkstyle/blob/master/chenjim_checks.xml)

## android studio（intellij）中安装使用
1. 要求STUDIO_JDK为jdk8以上,否则安装后无法正常使用，可以在环境变量中配置`STUDIO_JDK=../jdk`为studio的jdk版本
2. 打开studio，File-->setting-->Plugins-->Marketplace,搜索checkstyle，然后安装，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019060319070191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NTcWluZ2NoZW4=,size_16,color_FFFFFF,t_70)
3. 打开studio，File-->setting-->Plugins-->install plugin from disk，如下图，选择[官方的文件安装包](https://plugins.jetbrains.com/plugin/1065-checkstyle-idea/versions)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190603185957268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NTcWluZ2NoZW4=,size_16,color_FFFFFF,t_70)
4. 打开studio，File-->Settings-->OtherSettings-->CheckStyle，点“+”，选中“Use a local checkstyle file”，点击`Browse`，导入[简化版配置文件chenjim_checks.xml](https://gitee.com/chenjim/checkstyle/blob/master/chenjim_checks.xml)，然后**只勾选**刚导入的，点击`OK`，退出`Setting`
5. 打开studio，打开java代码，右键，点击`Check current file` 选项，检测当前代码是否有checkstyle异常，下方checkStyle栏目会显示检测结果.

本文地址 <http://blog.csdn.net/csqingchen/article/details/51896769>

## eclipse中安装使用
1. 菜单help-->install new software，在`work with`后面输入栏，输入在线安装地址 http://eclipse-cs.sourceforge.net/update，安装
2. 菜单Windows-->Preference，点击`checkstyle`，点击`New`，在`Check Configuration Properties`界面，`type`选择`External Configuration File`， 点击`Browse`，导入[chenjim_checks.xml](https://gitee.com/chenjim/checkstyle/blob/master/chenjim_checks.xml)，配置完成点击`OK`.
3. 选中刚新加的配置，点击右侧`Set as Default`
4. 打开一份java代码，右键，就有checkstyle选项

本文地址 <http://blog.csdn.net/csqingchen/article/details/51896769>

## 命令行中使用
>使用较繁琐，版本不一样还可能会出现其他异常，不推荐使用

1. 从[官方地址](https://sourceforge.net/projects/checkstyle/files/checkstyle/)或者 [Maven central](http://search.maven.org/#search|gav|1|g%3A%22com.puppycrawl.tools%22%20AND%20a%3A%22checkstyle%22)下载`checkstyle-**-all.jar`到自己的电脑
2. 使用命令形式如下
`java -jar checkstyle-**-all.jar的绝对路径 -c chenjim_checks.xml的绝对路径 要检测文件的路径`
例如
`java -jar /home/chenjim/bin/checkstyle/checkstyle-6.8.1-all.jar -c /home/chenjim/bin/checkstyle/chenjim_checks.xml ./MainActivity.java`
3. 命令太长了，太难记，太麻烦了，可以写一个脚本文件checkstyle/checkstyle.bat，里面内容如下
`java -jar /home/chenjim/bin/checkstyle/checkstyle-6.8.1-all.jar -c /home/chenjim/bin/checkstyle/chenjim_checks.xml $1`
将文件加到环境变量，加上可执行权限，然后`checkstyle ./MainActivity.java`就行了

本文地址 <http://blog.csdn.net/csqingchen/article/details/51896769>