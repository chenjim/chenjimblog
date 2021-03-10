## artifactory安装和使用

> 本文地址 <https://www.jianshu.com/p/ba57e23ddc1d>  

1. 下载 [artifactory-pro-6.6.0.zip](https://bintray.com/jfrog/artifactory-pro/download_file?file_path=org%2Fartifactory%2Fpro%2Fjfrog-artifactory-pro%2F6.6.0%2Fjfrog-artifactory-pro-6.6.0.zip )并解压  
   下载 [artifactory-injector-1.1.jar](https://gitee.com/chenjim/chenjimblog/tree/master/%E8%BD%AF%E4%BB%B6%E5%B7%A5%E5%85%B7/data/artifactory )

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

4. 启动 `.\artifactory-pro-6.6.0\bin\artifactory.bat start`
   出现 `Artifactory successfully started`表示启动成功  
   
5. 浏览器打开 <http://127.0.0.1:8081>进行相应的配置，包含输入步骤3生成的license   
   如需修改端口更改 `artifactory-pro-6.6.0\tomcat\conf\server.xml`中8081为相应的值即可 
   
6. 在`http://127.0.0.1:8081/artifactory/webapp/#/admin/repositories/remote` 添加一些maven镜像代理，比如：   
aliyun_public   https://maven.aliyun.com/repository/public   
aliyun_google   https://maven.aliyun.com/repository/google   
在`http://127.0.0.1:8081/artifactory/webapp/#/admin/repositories/virtual` 添加虚拟组   
比如 `repo_java` 组包含 `aliyun_public`  `aliyun_google`   
此时项目中所有maven{ *** } 都可以替换为`http://127.0.0.1:8081/artifactory/repo_java`   

本文地址 <https://www.jianshu.com/p/ba57e23ddc1d>  

参考文章  <https://www.jianshu.com/p/913a364d3245>  

