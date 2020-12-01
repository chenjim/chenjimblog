## artifactory安装和使用

> 相关的资源地址 [https://gitee.com/chenjim/artifactory](https://gitee.com/chenjim/artifactory)

1. 下载并解压zip包，linux/mac命令如下

   ` cat artifactory-pro-6.6.0.zip.00* > artifactory-pro-6.6.0.zip && unzip artifactory-pro-6.6.0.zip`

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

4. 启动 `.\artifactory-pro-6.6.0\bin\artifactory.bat start`
   出现 `Artifactory successfully started`表示启动成功
5. 浏览器打开 <http://127.0.0.1:8081>进行相应的配置，包含输入步骤3生成的license  
   如需修改端口更改 `artifactory-pro-6.6.0\tomcat\conf\server.xml`中8081为相应的值即可
6. 在`http://127.0.0.1:8081/artifactory/webapp/#/admin/repositories/remote`添加一些maven镜像代理  
比如：  
aliyun_public   https://maven.aliyun.com/repository/public  
aliyun_google   https://maven.aliyun.com/repository/google  
在`http://127.0.0.1:8081/artifactory/webapp/#/admin/repositories/virtual` 添加虚拟组  
比如 repo_java 组包含 aliyun_public  aliyun_google
此时项目中所有maven{*}  
都可以替换为`http://127.0.0.1:8081/artifactory/repo_java`

7. 更多内容可以参考 <https://www.jianshu.com/p/913a364d3245> 
