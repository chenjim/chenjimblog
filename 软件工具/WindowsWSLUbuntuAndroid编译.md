
> 本文首发地址：<https://blog.csdn.net/CSqingchen/article/details/123395496>  
> 最新更新地址：<https://gitee.com/chenjim/chenjimblog>

1. 以管理员身份运行 **命令提示符**，运行 `wsl --install`，  
   此过程会自动安装WSL、虚拟机，下载ubuntu20.04，  
   如果安装失败，可以尝试在 `Microsoft Store` 安装 ubuntu  
   若启动失败，可以重启一下试试    
   更多参考 <https://docs.microsoft.com/zh-cn/windows/wsl/install>  

2. 重启windows后，启动Ubuntu，自动安装  
   ![](https://pic.chenjim.com/202403242341167.png-blog)  
   从开始菜单启动，也可安装  

3. 替换换 ubuntu20.04 为阿里云源    
   `sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak`  
   `sudo vim /etc/apt/sources.list`  
   vim 执行如下命令替换  
   `:%s/archive.ubuntu.com/mirrors.aliyun.com/g`  
   `:%s/security.ubuntu.com/mirrors.aliyun.com/g`  
   `:%s/http/https/g`  
   更新已经安装的软件  
   `sudo apt update && sudo apt upgrade`   

4. Android编译依赖工具链  
   `sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig`  
   参考自  <https://source.android.com/setup/build/initializing>  
   以下按需安装
   ```
   sudo apt-get install libssl-dev libncurses5
   sudo apt-get install openjdk-8-jdk openjdk-8-jre
   sudo apt-get install git
   sudo apt-get install python3.7
   sudo apt-get install python2 (如果没有python2)
   sudo ln -s /usr/bin/python2 /usr/bin/python (如果repo命令报错没有python)
   sudo apt-get install libssl-dev libtinfo5 libncurses5 (编译 xr 4100 代码)
   sudo apt-get install libxml-simple-perl (编译 xr 5100 代码)
   ```
5. 迁移虚拟磁盘  
   Ubuntu 默认文件在C盘 `%USERPROFILE%\AppData\Local\Packages`名字含有 ubuntu 的目录（比如`C:\Users\chen\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_79rhkp1fndgsc\LocalState\rootfs`），磁盘空间有限，源码下载及编译需要约350G空间，  
   且不可以直接使用共享的Windows磁盘，迁移命令和说明如下    
   - 打开带管理员权限的PowerShell  
   ![](https://pic.chenjim.com/202403250056529.png-blog) 
   - 查看当前已经安装的WSL实例  
   `wsl -l -v`
   - 将现有WSL2实例备份导出  
   `wsl --export Ubuntu D:\Ubuntu_bak.tar`  
   - 注销现有实例了，‘Ubuntu’是上面步骤输出的实例名  
   `wsl --unregister Ubuntu`  
   - 将Ubuntu_bak.tar导入到指定存放虚拟磁盘镜像文件的路径‘D:\WSL_Ubuntu’  
   `wsl --import Ubuntu_new D:\WSL_Ubuntu D:\Ubuntu_bak.tar --version 2`  
   - 设置默认登录用户，`chen`之前Ubuntu配置的登录名   
   `ubuntu config --default-user chen`  
   如果安装的是 `Ubuntu-22.04`，需用用 `ubuntu2204 config --default-user chen`，其它类比    
   - 重新启动 wsl 登录即可  
  
   而且默认文件存在目录`%USERPROFILE%\AppData\Local\Packages`名字含有 ubuntu 的目录下，比如如下路径含有详细 ubuntu 文件，这样也是不够安全的  
  

   更多可以参考 <https://blog.csdn.net/u014175785/article/details/118181230>  
   如需给WSL添加虚拟磁盘，可参考  
   <https://blog.csdn.net/CSqingchen/article/details/126266773>  

6. 扩充wsl磁盘空间  
   wsl 默认的硬盘是256G，源码编译是不够的，需要扩充其默认大小  
   相关命令和说明如下   
   - 终止所有 WSL 实例  
     `wsl --shutdown`
   - 调整 WSL 2 VHD 的大小，**管理员权限**打开命令行提示符，执行以下命令  
   `diskpart`  
   `DISKPART> Select vdisk file="D:\WSL_Ubuntu\ext4.vhdx"`   
   注意路径是上面迁移磁盘后的路径，未迁移的默认路径如下  
   `%LOCALAPPDATA%\Packages\xxxx\LocalState\ext4.vhdx`  
   其中xxxx需要替换为以下命令的输出  
   `Get-AppxPackage -Name "*Ubuntu*" | Select PackageFamilyName`  
   - 当前虚拟值的最大值  
   `detail vdisk`  
   - 更改最大值，必须比原来大，以 MB 为单位，比如512G，如下  
   `expand vdisk maximum=512000`
   - 启动wsl，登录wsl，运行以下命令，让 WSL 知道它可扩展其文件系统的大小  
   `sudo mount -t devtmpfs none /dev`  
   `mount | grep ext4`       其中输出的 `/dev/sdX` 在下一步用  
   `sudo resize2fs /dev/sdb 512000M`    
   扩充完成   
   更多信息可以参考<https://docs.microsoft.com/zh-cn/windows/wsl/vhd-size>  

7. repo安装使用项目配置
   - 下载repo  
   `curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo && chmod +x repo`  
   - 添加repo到环境变量  
   `sudo mv repo /bin/repo`  
   - repo的运行过程中,会尝试访问 [官方的git源](https://gerrit.googlesource.com/git-repo) 更新自己，一般情况无法正常更新，  
   可以将如下内容复制到`~/.bashrc`最后，替换默认Repo源，再重启终端即可  
   `export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'`  
   更多可参考 <https://blog.csdn.net/CSqingchen/article/details/78125769>

8. 下载 [LineageOS](https://github.com/LineageOS/) 源码  
`repo init -u https://github.com/LineageOS/android.git -b lineage-19.1`  
`repo sync`  
如果一定要Google Android 源码，可以参考 <https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/>  


9. 源码编译  
`source build/envsetup.sh`  
`lunch lineage****-userdebug`  
`make -j$(nproc) 2>&1 | tee build.log `   
或者使用脚本  `brunch your_device_codename` 


10.  设置windows目录大小写敏感  
`fsutil.exe file SetCaseSensitiveInfo D:\code enable`   
![](https://pic.chenjim.com/202403250055062.png-blog)

----------

相关文章连接   
[Window 10 使用WSL2下载编译Android 10 系统源码](https://blog.csdn.net/baidu_24392053/article/details/119081623)  
[how-to-build-lineageos-on-windows-10-using-wsl-2](https://www.xda-developers.com/how-to-build-lineageos-on-windows-10-using-wsl-2/)  

