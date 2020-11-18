@[toc]
## repo安装  
> 首先确认电脑是否已经安装repo，执行`which repo`找不到repo时需要安装  
> 本文链接  <http://blog.csdn.net/CSqingchen/article/details/78125769>  

1. 下载repo  
`git clone https://github.com/android/tools_repo /git-repo`  

2. 添加repo到环境变量  
`sudo ln -s /git-repo/repo /bin/repo`  

3. 修改/git-repo/repo中REPO_URL后面的地址  
将`REPO_URL = 'https://gerrit.googlesource.com/git-repo'`  
替换为  
`REPO_URL = 'https://github.com/android/tools_repo'`  

## repo使用  
> repo 说明文档参见 `repo help`，repo管理git项目，对于任意一个被管理的git项目，可以使用git相关的命令进行版本控制  
> 本文链接 <http://blog.csdn.net/CSqingchen/article/details/78125769>  
> Android 9.0.0_r60 的manifest [源码@google](https://android.googlesource.com/platform/manifest/+/refs/tags/android-9.0.0_r60/default.xml) 或 [源码@gitee](https://gitee.com/chenjimcom/repoABC/blob/9.0.0_r60/default.xml)  

1. 初始化  
`mkdir repoABC&&cd repoABC`  
`repo init -u https://gitee.com/chenjimcom/repoABC`  

2. 同步repo管理的git仓库代码  
`repo sync`  
强制同步整个仓库  
`repo sync --force-sync`  

3. 将所有repo控制的git仓库远程master分支checkout   
`repo forall -c git checkout -b master remotes/origin/master`  
清除还原本地的修改  
`repo forall -c 'git clean -df;git checkout .'`  

4. 将某个时间段所有仓库修改记录显示出来  
`repo forall -c git log  --name-status --since="2016-09-13" --until="2017-02-01"`  

5.  查看工程中所有仓库的修改状态（包括文件位置）  
`repo status`  

更多参考文章
1. [android repo中manifest.xml的详解](http://blog.csdn.net/shift_wwx/article/details/19557031)
2. [怎么针对自己项目工程建立Repo管理多个git仓库](https://www.zhihu.com/question/41440585)
3. [repo的使用小结博客](http://blog.csdn.net/wh_19910525/article/details/8164107)