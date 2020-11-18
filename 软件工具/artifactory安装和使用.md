
[TOC]

> 本文首发地址 <https://h89.cn/archives/7.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  

### artifactory-pro-6.6.0 安装使用

1. 下载 artifactory-pro-6.6.0.zip 并解压，如需文件，邮件到 <me@h89.cn>  
   下载 artifactory-injector-1.1.jar 如需文件，邮件到 <me@h89.cn>  
2. 绿化处理，命令`java -jar artifactory-injector-1.1.jar`，选择2，然后需要输入artifactory解压后的目录，详细如下

   ```bash
   $ java -jar artifactory-injector-1.1.jar
   What do you want to do?
   1 - generate License String
   2 - inject artifactory
   exit - exit
   2
   where is artifactory home? ("back" for back)
   D:\artifactory\artifactory-pro-6.6.0/
   artifactory detected. continue? (yes/no)
   yes
   putting another WEB-INF/lib/artifactory-addons-manager-6.6.0.jar
   META-INF/
   org/
   org/jfrog/
   ...
   
   ```

3. 生成授权License，选择1，记录生成的license，然后exit退出

   ```bash
   What do you want to do?
   1 - generate License String
   2 - inject artifactory
   exit - exit
   1
   eyJhcnRpZmFjdG9yeSI6eyJ......ydGllcyI6e319fQ==
   
   ```

   **备注** 
   - 如果没有2、3的处理，不能使用全部功能，跟社区版本差不多，只能添加部分镜像
   - 此处的license在后续配置中会使用到，需要保存一下
   
4. 启动 `.\artifactory-pro-6.6.0\bin\artifactory.bat start`  
   Linux:`./artifactory-pro-6.6.0/bin/artifactory.sh`  
   `ps -ef | grep artifactory` Linux下查看启动的进程信息   
   出现 `Artifactory successfully started`表示启动成功  

5. 建议修改 `artifactory-pro-6.6.0\tomcat\conf\server.xml` 中 端口 `8081` 为`18081`，`server.xml`中的其它端口建议同步修改，避免冲突引起启动异常  

6. 浏览器打开 <http://192.168.3.242:18081> 进行相应的配置，包含输入步骤3生成的license   
   如需修改端口更改 为相应的值即可，如18081，如果本机还有服务使用此文件的其它端口，注意更改。 
   
7. 菜单 `Admin`  ->  `Repositories`  -> `Remote`  -> `NEW` 选择 `maven`  
   添加如下镜像代理    
   | Repository Key | URL |
   |--|--|
   | aliyun_public  |   https://maven.aliyun.com/repository/public   |
   | aliyun_google  |   https://maven.aliyun.com/repository/google   |
   | jitpack        |   https://jitpack.io                      |  


8. 菜单 `Admin`  ->  `Repositories`  -> `Virtual`  -> `NEW` 选择 `maven`  
   `Repository Key` 填写 `android`   
   `Repositories`  栏 `Selected Repositories`  选择上面添加的 `aliyun_public`、`aliyun_google`、`jitpack`等    

9. 此时项目中所有 `maven{ *** }` 都可以替换为
   ``` groovy 
      maven {
         url 'http://192.168.3.242:18081/artifactory/android'
         //gradle 7.0+  需要打开以下
         //allowInsecureProtocol = true
      }
   ```

> 本文地址 <https://gitee.com/chenjim/chenjimblog>  
---

### artifactory 社区版安装使用


1. 下载地址 <https://jfrog.com/open-source/>    

2. 安装使用说明   
<https://www.jfrog.com/confluence/display/JFROG/Installing+Artifactory>  
完整的功能列表请参考：  
<https://www.jfrog.com/confluence/display/JCR/Overview>  

3. 默认用户名 admin 密码 password   

---

### 代理 Gradle 本地依赖代理
>Gradle项目中下载首次编译，需要从 `https://services.gradle.org/distributions` 下载 `gradle-*.zip`
>可能会比较慢，我们可以用 artifactory 做代理  

#### 方案一 
1. 菜单 `Admin`  ->  `Repositories`  -> `Remote`  -> `NEW`   
2. 弹出对话框 `Select Package Type`, 选择 `Generic`   
3. `Repository Key` 填写 `distributions` ，`URL` 填写 `https://services.gradle.org/distributions`   
5. 保存即可  

#### 方案二
1. 菜单 `Admin` --> `Repositories` --> `Local`  
2. 选择 `New`，选择 `maven`，输入 `Repository Key` 内容为 `distributions`
3. 菜单 `Artifacts` 选择上一步添加的 `distributions`，再点右上角 `Deploy` 
4. 在弹出界面上传下载的 `gradle-*.zip` 即可


以上两个方案都可以，网络见到的都是第二种，第一种相对简单，不用每次上传本地的 gradle-*.zip    

最后替换 `gradle/wrapper/gradle-wrapper.properties` 中     
`distributionUrl=https\://services.gradle.org/distributions/gradle-7.0.2-all.zip`   为    
`distributionUrl=http\://192.168.3.242\:18081/artifactory/distributions/gradle-7.0.2-all.zip`  
即可   


----

### 安卓上传 aar 到 artifactory   
1. `Baselibrary/build.gradle`同目录添加文件 `artifactoryPublish.gradle`，内容如下   
```groovy
apply plugin: 'maven-publish'
apply plugin: 'com.jfrog.artifactory'
//定义artifactory仓库的地址，按照你自己的修改
def MAVEN_LOCAL_PATH = 'http://192.168.3.242:18081/artifactory'
// 定义构件的相关信息
// 当其他项目远程依赖该构件的时候，结构类似就是 implementation 'GROUP_ID:ARTIFACT_ID:VERSION_NAME'
def GROUP_ID = 'com.chenjim.android'
def ARTIFACT_ID = 'andlibs'
def VERSION_NAME = '0.0.1'
publishing {
    publications {
        aar_pub(MavenPublication) {//注意这里定义的 aar_pub，在artifactoryPublish 下需要使用
            groupId = GROUP_ID
            artifactId = ARTIFACT_ID
            version = VERSION_NAME
            // aar文件所在的位置
            // module打包后所在路径为module模块下的build/outputs/aar，生成的aar名称为：module名-release.aar
            artifact("$buildDir/outputs/aar/${project.getName()}-debug.aar")
            //当有其他 dependencies api 时可能需要以下
            //参考自 https://chowdera.com/2021/03/20210305112748155c.html
//            pom.withXml {
//                def dependencies = asNode().appendNode("dependencies")
//                configurations.api.allDependencies.each {
//                    def dependency = dependencies.appendNode("dependency")
//                    dependency.appendNode("groupId", it.group)
//                    dependency.appendNode("artifactId", it.name)
//                    dependency.appendNode("version", it.version)
//                }
//            }
        }
    }
}
artifactoryPublish {
    contextUrl = MAVEN_LOCAL_PATH
    publications('aar_pub')        //注意这里使用的是上面定义的 aar_pub
   //需要提前创建 Local Repositories,名为 jx_android   
    clientConfig.publisher.repoKey = 'j_android'        //上传到的仓库地址
    clientConfig.publisher.username = 'j_jfrog'        //artifactory 登录的用户名
    clientConfig.publisher.password = 'djofYSKEWC2vPh1'    //artifactory 登录的密码
}
```
2. `Baselibrary/build.gradle` 中添加  
   `apply from: "./artifactoryPublish.gradle"`
3. 根目录 `build.gradle` 中 `dependencies` 节点添加  
   `classpath "org.jfrog.buildinfo:build-info-extractor-gradle:4.15.2"`

> 本文地址 <https://gitee.com/chenjim/chenjimblog>  

---

### 用sed脚本替换安卓依赖 
将以下内容粘贴到`sed_artifactory.sh`，加到环境变量，在需要的目录执行即可  
脚本用到 `sed` 和 `grep` ，windows 下需要安装 `Git` 在 `Git Bash`执行  
``` bash
#!/usr/bin/env bash

# 自动替换Android依赖为私有maven脚本

#set -x

# 替换"repositories {"为
# "repositories {
#          maven {
#              url 'http://192.168.3.242:18081/artifactory/android'
#              //allowInsecureProtocol = true
#          }
# "         
IN_DATA="repositories {"
OUT_DATA="repositories {\r\n        maven {\r\n             url 'http:\/\/192.168.3.242:18081\/artifactory\/android'\r\n             \/\/ allowInsecureProtocol = true\r\n        }" 
sed -i "s/$IN_DATA/$OUT_DATA/g" `grep "$IN_DATA" -rl ./`

#替换gradle-wrapper.properties中gradle地址  
GI="https\\\:\/\/services.gradle.org\/distributions"
GO="http:\/\/192.168.3.242:18081\/artifactory\/distributions"
sed -i "s/$GI/$GO/g" `grep "$GI" -rl ./`

```

参考自  
<https://blog.csdn.net/u010976213/article/details/105710511>   
<https://chowdera.com/2021/03/20210305112748155c.html>  


---

### 用artifactory做npm本地镜像

1. 菜单 `Admin` --> `Repositories` --> `Remote` -->  `NEW`  
2. 选择 `npm` 
3. `Repository Key` 填写 `npm` , `URL` 填写  `https://registry.npm.taobao.org`  
   经过如上修改，我们就可以使用artifactory做npm的镜像了  
4. 用淘宝镜像配置为 `npm config set registry https://registry.npm.taobao.org`  
   此配置可以替换为  `npm config set registry http://192.168.3.242:18081/artifactory/npm`  


---

### artifactory 上传文件大小配置  

1. 菜单 `Admin` --> `GeneralConfiguration`  
2. 更改 `File Upload Max Size` 内容即可


----

### artifactory无法访问，提示401授权异常   
1. 菜单 `Admin`  --> `Security Configuration`  
2. 勾选 ` Allow Anonymous Access` ,早期的版本已默认勾选，新版本可能没有


----
> 本文地址 <https://gitee.com/chenjim/chenjimblog>  



