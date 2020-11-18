[TOC]

> 本文首发地址 <http://blog.csdn.net/CSqingchen/article/details/78125769>

## repo安装
> 首先确认电脑是否已经安装repo，执行`which repo`找不到repo时需要安装  


1. 下载repo  
   `curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo && chmod +x repo`

2. 添加repo到环境变量  
   `sudo mv repo /bin/repo`

3. repo的运行过程中,会尝试访问[官方的git源](https://gerrit.googlesource.com/git-repo )更新自己,一般情况无法正常更新，  
   可以将如下内容复制到你的~/.bashrc里，并重启终端模拟器  
   `export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'`

参考自 **Git Repo 镜像使用帮助** <https://mirrors.tuna.tsinghua.edu.cn/help/git-repo>

## repo 配置  

参考 Android 9.0.0_r60 的 manifest:  
<https://android.googlesource.com/platform/manifest/+/refs/tags/android-9.0.0_r60/default.xml>  
或者  
<https://gitee.com/chenjim/repoABC/blob/9.0.0_r60/default.xml>  

安卓源码访问可以[ 使用工具 ](https://suying555.net/auth/register?code=8ogy)


## repo 使用
> repo 说明文档参见 `repo help`，repo管理git项目，对于任意一个被管理的git项目，可以使用git相关的命令进行版本控制  
> 本文链接 <http://blog.csdn.net/CSqingchen/article/details/78125769>

1. 初始化  
   `mkdir repoABC&&cd repoABC`  
   `repo init -u https://gitee.com/chenjim/repoABC`

2. 同步repo管理的git仓库代码  
   `repo sync`  
   可以看到repoABC中配置的仓库已经同步到了当前目录

3. 将所有repo控制的git仓库远程master分支checkout  
   `repo forall -c git checkout -b master remotes/origin/master`

4. 将某个时间段所有仓库修改记录显示出来  
   `repo forall -c git log  --name-status --since="2016-09-13" --until="2017-02-01"`

5.  查看工程中所有仓库的修改状态（包括文件位置）  
    `repo status`

参考文档
- [Android Repo 命令参考资料](https://source.android.google.cn/setup/develop/repo?hl=zh-cn)
- [android repo中manifest.xml的详解](http://blog.csdn.net/shift_wwx/article/details/19557031)
- [怎么针对自己项目工程建立Repo管理多个git仓库](https://www.zhihu.com/question/41440585)
- [repo的使用小结博客](http://blog.csdn.net/wh_19910525/article/details/8164107)  