@[toc]
### Git配置和常用命令
> 下载地址 [https://git-scm.com/downloads](https://git-scm.com/downloads)

#### 初始配置
```
git config --global user.name jim.chen
git config --global user.email me@h89.cn

git config --global alias.cp cherry-pick &&
git config --global alias.co checkout &&
git config --global alias.ci commit &&
git config --global alias.br branch &&
git config --global alias.st status &&
git config --global core.editor vi

warning: LF will be replaced by CRLF
mac/linux:  
git config --global core.autocrlf input
windows:  
git config --global core.autocrlf true（默认情况）
//详细配置说明 https://www.jianshu.com/p/0a747b2b76a2
```

```
//代理配置
全局git代理
git config --global http.proxy socks5://127.0.0.1:1080  
github.com 使用代理
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
移除代理配置
git config --global --unset http.proxy
git config --global --unset http.https://github.com.proxy
```

```
# 生成ssh的key命令  
ssh-keygen  直接回车

# 查看当前用户的配置  
vi ~/.gitconfig   
git config -l  

# 长期存储密码  
git config --global credential.helper store  
或者  
git remote rm origin  
git remote add origin  http://yourname:password@git.oschina.net/name/project.git  
```
---

#### 创建commit模板
```
1. 建立~/.git/template文件
OverView: （简单的修改描述）
Bug ID: （项目名称，bug号：例如name12345，添加新功能bug为0）
Description: （bug标题，修改的详细描述）
2. 设置模板
git config --global commit.template ~/.git/template
3. 设置文本编辑器
git config --global core.editor vi
4. gerrit commit  提交
git commit -s 
#(会自动打开模板，填好save就行了，git会自动提交)

gerrit 提交到master  
git push origin HEAD:refs/for/master

gerrit  审核不通过，再次commit  
git commit -s --amend


# 对于git项目为了在提交时自动产生change-id，  
scp -P 29418 -p  username@10.1.11.10:/hooks/commit-msg .git/hooks
scp -p -P 29418  chenjim@review.putao.io:hooks/commit-msg .git/hooks/
```
---

#### 建立配置ssh的配置文件
```
$ cd ~/.ssh
$ touch config
$ vi config
### 再输入以下内容
Host ha
HostName review.putao.io
User chenjim
Port 29418
IdentityFile ~/.ssh/id_rsa
```
```
##然后，下面写法结果等效
git clone ssh://chenjim@review.putao.io:29418/PTlauncher  
git clone ssh://ha/PTlauncher

repo init -u ssh://ha/t700_v5.1/manifest
repo sync
```
---

#### 创建新git仓库并push到远程:
```
mkdir GitCamera
cd GitCamera
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin  https://git.oschina.net/chenjim/GitCamera.git
git push -u origin master
```
---

#### push已有项目到远程仓库
```
cd existing_git_repo
git remote add origin  https://git.oschina.net/chenjim/GitCamera.git
git push -u origin master
```
---

#### git stash 的使用
```
git stash  缓存当前所有修改
git stash list 列出当前所有的缓存
git stash pop 应用最近一次的缓存，并删除当前缓存
git stash show -p stash@{0} 显示第0个缓存的详细修改，无-p，只显示修改的文件列表
git stash clear 清除所有的缓存
```
---

#### 查看、添加、提交、删除、找回，重置修改文件
```
git help # 显示command的help
git show # 显示某次提交的内容 git show $id
git co -- # 抛弃工作区修改
git co . # 抛弃工作区修改
git add # 将工作文件修改提交到本地暂存区
git add . # 将所有修改过的工作文件提交暂存区
git rm # 从版本库中删除文件
git rm --cached # 从版本库中删除文件，但不删除文件
git reset # 从暂存区恢复到工作文件
git reset -- . # 从暂存区恢复到工作文件
git reset --hard # 恢复最近一次提交过的状态，即放弃上次提交后的所有本次修改
git ci git ci . git ci -a # 将git add, git rm和git ci等操作都合并在一起做　　
git ci -am "some comments"
git ci --amend # 修改最后一次提交记录
git revert <$id> # 恢复某次提交的状态，恢复动作本身也创建次提交对象
git revert HEAD # 恢复最后一次提交的状态
git reset --hard HEAD^ 销毁最后一次本地提交
git push -f origin HEAD^:master  销毁最后一次远程提交
```
---

#### 查看文件diff
```
git diff # 比较当前文件和暂存区文件差异 git diff
git diff < id1>< id2> # 比较两次提交之间的差异
git diff .. # 在两个分支之间比较
git diff --staged # 比较暂存区和版本库差异
git diff --cached # 比较暂存区和版本库差异
git diff --stat # 仅仅比较统计信息
```
---

#### 查看提交记录
```
git log # 查看该文件每次提交记录
git log -p # 查看每次详细修改内容的diff
git log -p -2 # 查看最近两次详细修改内容的diff
git log --stat #查看提交统计信息
git log --stat --since=3.weeks #查看最近三周提交统计信息
git log --stat --since="2018-06-02" #查看日期"2018-06-02"之后的提交统计信息
git log --pretty=oneline -4  # 显示最近4次修改记录，包含修改内容
git log -p -4 # 显示最近4次修改记录，包含修改内容
git log -4 # 显示最近4次修改记录，只主要信息，不包含修改内容
git log -3 --name-status  #显示最后4次修改牵扯到的文件
git log --author="jim.chen"  #显示某个作者的log 
git show da28c5581 #显示某一次修改的内容  
```
---

#### Git 本地分支管理--查看、切换、创建和删除分支
```
git br --all #查看所有本地和远程分支名称
git br -r # 查看远程分支
git br dev# 创建新的分支dev
git br -v # 查看各个分支最后提交信息
git br --merged # 查看已经被合并到当前分支的分支
git br --no-merged # 查看尚未被合并到当前分支的分支
git co dev # 切换到一创建的dev分支
git co -b dev # 基于当前分支，创建新的分支dev，并且切换过去
git co $id # 把某次历史提交记录checkout出来，但无分支信息，切换到其他分支会自动删除
git co -b dev origin/master 把远程的master迁出到本地分支dev
git co -b debug 0788dee 把某VI提交0788dee迁出到本地分支debug
git br -d # 删除某个分支
git br -D # 强制删除某个分支 (未被合并的分支被删除的时候需要强制)

#分支 master 设置为跟踪来自 origin 的远程分支 master。
git branch --set-upstream-to=origin/master master
or
git branch --set-upstream master origin/master
git checkout -b master remotes/origin/master

#当前HEAD所在分支的名称
git rev-parse --abbrev-ref HEAD
```
---

#### 分支合并和rebase
```
git merge # 将branch分支合并到当前分支
git merge origin/master --no-ff # 不要Fast-Foward合并，这样可以生成merge提交
git rebase master # 将master rebase到branch，相当于： git co && git rebase master && git co master && git merge
 Git补丁管理(方便在多台机器上开发同步时用)
git diff > ../sync.patch # 生成补丁
git apply ../sync.patch # 打补丁
git apply --check ../sync.patch #测试补丁能否成功
patch -p1 < ../sync.patch #使用patch命令大补丁
```
---


#### Git暂存管理
```
git stash # 暂存
git stash list # 列所有stash
git stash apply # 恢复暂存的内容
git stash drop # 删除暂存区
```
---

#### Git远程分支管理
```
git pull # 抓取远程仓库所有分支更新并合并到本地
git pull --no-ff # 抓取远程仓库所有分支更新并合并到本地，不要快进合并
git pull --rebase # 抓取远程分支到本地，将本地修改rebase远程最新后
git fetch origin # 抓取远程仓库更新
git merge origin/master # 将远程主分支合并到本地当前分支
git push # push所有分支
git push origin master # 将本地主分支推到远程主分支
git push -u origin master # 将本地主分支推到远程(如无远程主分支则创建，用于初始化远程仓库)
git push origin # 创建远程分支， origin是远程仓库名
git push origin : # 创建远程分支
git push origin :dev #先删除本地分支dev(git branch -r -d origin/dev)，然后再push删除远程分支dev
```
---

#### Git远程仓库管理
```
git remote -v # 查看远程服务器地址和仓库名称
git remote show origin # 查看远程服务器仓库状态
git remote add origin git@gitee.com:chenjim/thirdPartyJniSo.git # 添加远程仓库地址
git remote set-url origin git@gitee.com:chenjim/thirdPartyJniSo.git # 设置远程仓库地址(用于修改远程仓库地址) 
git remote rm # 删除远程仓库
git remote prune origin 删除已删除的分支(同步本地远程分支)
```
---

#### 创建远程仓库
```
git clone --bare robbin_site robbin_site.git # 用带版本的项目创建纯版本仓库
scp -r my_project.git git@ git.csdn.net:~ # 将纯仓库上传到服务器上
mkdir robbin_site.git && cd robbin_site.git && git --bare init # 在服务器创建纯仓库
git remote add origin git@gitee.com:chenjim/thirdPartyJniSo.git # 设置远程仓库地址
git push -u origin master # 客户端首次提交
git push -u origin develop # 首次将本地develop分支提交到远程develop分支，并且track
git remote set-head origin master # 设置远程仓库的HEAD指向master分支
```
---

#### tag 使用
```
#创建带有说明的标签，用-a指定标签名，-m指定说明文字：
git tag -a V0.1 -m "version 0.1 released push url" d5a65e9
git show V0.1 #查看标签信息
git tag V1.0         #给当前commit添加tag
git tag V1.0 471fd27 #给指定commit 471fd27 添加tag
git push origin V1.0 #将指定tag推送到远程
git push --tags        #将所有tag推送到远程
git push origin --tags #将所有tag推送到远程
git tag -d V1.0   #删除本地tag
git push origin :V1.0             #删除远程tag
git push origin --delete tag V1.0 #删除远程tag
git ls-remote --tags origin  #查询远程tags的命令如下：
```
[git 如何同步本地tag与远程tag](https://blog.csdn.net/wei371522/article/details/83186077)
问题场景：  
同事A在本地创建tagA并push同步到了远程->同事B在本地拉取了远程tagA(git fetch)->同事A工作需要将远程标签tagA删除->同事B用git fetch同步远端信息，git tag后发现本地仍然记录有tagA  

分析：对于远程repository中已经删除了的tag，即使使用git fetch --prune，甚至"git fetch --tags"确保下载所有tags，也不会让其在本地也将其删除的。而且，似乎git目前也没有提供一个直接的命令和参数选项可以删除本地的在远程已经不存在的tag（我目前是没找到有关这类tag问题的git命令~~，有知道的同学可以告知我下，互相进步）。  
解决方法：
git tag -l | xargs git tag -d   #删除所有本地分支  
git fetch origin --prune  #从远程拉取所有信息  

---

#### reflog 使用
>reflog是Git操作的一道安全保障，它能够记录几乎所有本地仓库的改变。包括所有分支commit提交，已经删除（其实并未被实际删除）commit都会被记录。总结：只要HEAD发生变化，就可以通过reflog查看到。
```
git reflog #查看最近的git变化
git checkout HEAD@{90}   #将指定的变化迁出

```
---


**相关链接**
-  [安卓软件开发常用命令集合](https://blog.csdn.net/CSqingchen/article/details/108150595)