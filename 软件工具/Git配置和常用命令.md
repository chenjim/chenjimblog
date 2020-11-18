
[TOC]

> Git下载地址 <https://git-scm.com/downloads>    
> 本文首发地址 <https://h89.cn/archives/4.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  

## 初始配置

1. 账号邮箱配置

   `git config --global user.name chenjim`  
   `git config --global user.email me@h89.cn`

2. alias简写配置
   ```
   git config --global alias.cp cherry-pick
   git config --global alias.co checkout
   git config --global alias.ci commit
   git config --global alias.br branch
   git config --global alias.st status
     ```

3. warning: LF will be replaced by CRLF  
   mac/linux:  
   `git config --global core.autocrlf input`
   windows:  
   `git config --global core.autocrlf true`（默认情况） 详细配置说明 <https://www.jianshu.com/p/0a747b2b76a2>

4. Git Bash 中文名称、路径变成xx%，显示八进制编码，解决方法：   
   `git config --global core.quotepath false`

5. git代理配置
   - 全局git代理配置   
   `git config --global http.proxy socks5://127.0.0.1:1080`
   - 只github.com 使用代理
   `git config --global http.https://github.com.proxy socks5://127.0.0.1:1080`
   - 移除代理配置  
   `git config --global --unset http.proxy`
   `git config --global --unset http.https://github.com.proxy`

6. 生成ssh的key命令  
   `ssh-keygen`  直接回车

7. 查看当前用户的配置  
   `vi ~/.gitconfig`   或者`git config -l`

8. 长期存储密码  
   `git config --global credential.helper manager` 
   默认，Windows存储在 `控制面板\用户帐户\凭据管理器`  
   `git config --global credential.helper store`  
   最终密码明文存储在文件 `~/.git-credentials`(`C:\Users\admin\.git-credentials`)，并不安全    
   或者将账号密码存储在 http url  
   `git remote rm origin`  
   `git remote add origin  http://yourname:password@git.oschina.net/name/project.git`

9. 设置文本编辑器  
   `git config --global core.editor vi`

10. 配置是否忽略文件权限  
    `git config --global core.filemode false `  
    windows下：false不忽略可执行权限；true忽略。Linux下相反。  	
    windows下对文件`gradlew`增加可执行权限     
    `git add --chmod=+x gradlew`   增加`+x`;移除`-x`  
    或者`git update-index --chmod=+x gradlew`

11. windows 不忽略文件大小写，默认忽略大小写。  
    `git config --global core.ignorecase false`
    `git config core.ignorecase false`

12. 全局文件忽略  
    增加文件 `~/.gitignore_gb` , 参考 `.gitignore` 配置要忽略的内容  
    最后执行 `git config --global core.excludesFile ~/.gitignore_gb`   

---

## 创建commit模板

1. 建立`~/.git/template`文件,内容如下

```
    OverView: （简单的修改描述）
    Bug ID: （项目名称，bug号：例如name12345，添加新功能bug为0）
    Description: （bug标题，修改的详细描述）
```

2. 设置模板  
   `git config --global commit.template ~/.git/template`

3. gerrit commit 提交  
   `git commit -s`  会自动打开模板，填好save就行了，git会自动提交
  - gerrit 提交到master  
    `git push origin HEAD:refs/for/master`
  - gerrit 审核不通过，再次commit  
    `git commit -s --amend`
  - 对于git项目为了在提交时自动产生change-id，  
    `scp -P 29418 -p chenjim@review.chenjim.io:hooks/commit-msg .git/hooks/`

---

## ssh的配置文件的使用

- 创建配置文件
  ```
  $ cd ~/.ssh
  $ touch config
  $ vi config
  ```
- 输入以下文件内容
  ```
  Host ha
  HostName review.chenjim.io
  User chenjim
  Port 29418
  IdentityFile ~/.ssh/id_rsa
  ```
- 完成后，下面写法结果等效  
  `git clone ssh://chenjim@review.chenjim.io:29418/DemoApp`  
  `git clone ssh://ha/DemoApp`

---

## 创建新git仓库并push到远程:

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
  在`~/.bashrc` 添加 `gl` 支持  
  `alias gl="git log --oneline --all --graph --decorate"`  


---

## push已有项目到远程仓库

  ```
  cd existing_git_repo
  git remote add origin  https://git.oschina.net/chenjim/GitCamera.git
  git push -u origin master
  ```

---

## 查看、添加、提交、删除、找回，重置修改文件

`git help` 显示command的help  
`git show` 显示最后一次提交的内容  
`git show de0d8f066ace` 显示commit id 为 de0d8f066ace 的提交修改的内容  
`git co .` 抛弃工作区修改  
`git add .` 将所有修改过的工作文件提交暂存区  
`git rm abc/abc.java` 从版本库中删除文件  
`git reset abc/abc.java` 从暂存区恢复到工作文件  
`git reset .` 从暂存区恢复到工作文件  
`git reset --hard` 恢复最近一次提交过的状态，即放弃上次提交后的所有本次修改  
`git reset HEAD~1 --hard` 销毁最后一次本地提交  
`git ci -am "some comments"` 将git add, git rm和git ci等操作都合并在一起  
`git ci --amend` 修改最后一次提交记录  
`git revert <$id>` 还原某次提交，会创建一个新的commit  
`git revert HEAD` 恢复最后一次提交,会创建一个新的commit  
`git filter-branch --force --index-filter 'git rm -r --cached --ignore-unmatch path-to-your-remove-file' --prune-empty --tag-name-filter cat -- --all`  
永久删除文件(包括历史记录)，[path-to-your-remove-file](https://www.cnblogs.com/shines77/p/3460274.html) 是要删除的文件的相对路径  

---

## git stash 的使用

`git stash`  缓存当前所有修改  
`git stash list` 列出当前所有的缓存  
`git stash pop` 应用最近一次的缓存，并删除当前缓存  
`git stash show -p stash@{0}` 显示第0个缓存的详细修改，无-p，只显示修改的文件列表  
`git stash clear` 清除所有的缓存

---

## 查看文件diff

`git diff`  比较当前文件和暂存区文件差异  
`git diff <id1> <id2>` 比较两次提交之间的差异  
`git diff dev main` 在两个分支之间比较  
`git diff --staged` 比较暂存区和版本库差异  
`git diff --stat HEAD~1` 仅比较跟上一次修改的统计信息

---

## 查看提交记录

`git log .` 查看当前目录所有的提交记录  
`git log --stat` 查看提交统计信息  
`git log --stat --since=3.weeks` 查看最近三周提交统计信息  
`git log --stat --since="2018-06-02"` 查看日期"2018-06-02"之后的提交统计信息  
`git log --pretty=oneline -4`   显示最近4次修改记录，包含修改内容  
`git log -p da28c5581` 显示某次详细修改内容  
`git log -p -4` 显示最近4次修改记录，包含修改内容  
`git log -4 ` 显示最近4次修改记录，只主要信息，不包含修改内容  
`git log -4 --name-status`  显示最后4次修改牵扯到的文件  
`git log --author="jim.chen"`  显示某个作者的log   
`git show da28c5581` 显示某某次修改的内容

---

## Git 本地分支管理--查看、切换、创建和删除分支

**注意**：`br` 为 branch 的 alias 简写  
`git br --all` 查看所有本地和远程分支名称  
`git br -r` 查看远程分支  
`git br dev` 基于当前HEAD创建新的分支dev  
`git br -v`  查看各个分支最后提交信息  
`git br --merged`  查看已经被合并到当前分支的分支  
`git br --no-merged`  查看尚未被合并到当前分支的分支  
`git co dev`  切换到已经存在的dev分支  
`git co -b dev`  基于当前HEAD，创建新的分支dev，并且切换到新的分支  
`git co $id`  切换代码到某次提交记录，无分支信息，切换到其他分支会自动删除  
`git co -b dev origin/master` 把远程的master迁出到本地分支dev  
`git co -b debug 0788dee` 基于提交0788dee切换到新的debug分支  
`git br -d dev` 删除dev本地分支  
`git br -D dev`  强制删除dev分支 (未被合并的分支被删除的时候需要强制)

分支 master 设置为跟踪来自 origin 的远程分支 master。    
`git branch --set-upstream-to=origin/master master`  
或者   
`git branch --set-upstream master origin/master`  
`git checkout -b master remotes/origin/master`

当前HEAD所在分支的名称  
`git rev-parse --abbrev-ref HEAD`

---

## 分支合并和rebase

`git merge dev`  将dev分支合并到当前分支  
`git merge origin/main --no-ff`  不要Fast-Foward合并远程main分支，这样可以生成merge提交  
`git rebase master`  将当前分支 rebase 到master节点之后  
Git补丁管理(方便在多台机器上开发同步时用)  
`git diff > ../sync.patch`  生成补丁  
`git apply ../sync.patch`  使用补丁  
`git apply --check ../sync.patch` 测试补丁能否成功  
`patch -p1 < ../sync.patch` 使用`patch`命令打补丁

---

## Git暂存管理

`git stash`  暂存  
`git stash pop` 恢复暂存的内容  
`git stash list`  列所有stash  
`git stash apply`  恢复暂存的内容  
`git stash drop`  删除暂存区  
`git stash clear`  清除所有暂存

---

## Git远程分支管理

`git pull`  抓取远程仓库所有分支更新并合并到本地  
`git pull --no-ff` 抓取远程仓库所有分支更新并合并到本地，不要快进合并  
`git pull --rebase`  抓取远程分支到本地，将本地修改rebase远程最新后  
`git fetch origin`  抓取远程仓库更新  
`git merge origin/master`  将远程主分支合并到本地当前分支  
`git push  push` 所有分支  
`git push origin dev`  将本地dev分支推到远程dev分支  
`git push -u origin main`  将本地main分支推到远程(如无远程主分支则创建，用于初始化远程仓库)  
`git push origin :dev`  删除远程分支dev  
`git push github HEAD:dev --force` 将当前HEAD强制推送到dev分支，**github** 参见下一节 'Git远程仓库管理'

---

## Git远程仓库管理

`git remote -v`  查看远程服务器地址和仓库名称  
`git remote show origin`  查看远程服务器仓库状态  
`git remote add origin git@gitee.com:chenjim/thirdPartyJniSo.git`  添加远程仓库地址  
`git remote add github git@github.com:chenjim/thirdPartyJniSo.git`  添加远程仓库地址  
`git remote set-url origin git@gitee.com:chenjim/thirdPartyJniSo.git`  设置远程仓库地址(用于修改远程仓库地址)   
`git remote rm`  删除远程仓库地址  
`git remote prune origin` 删除已删除的分支(同步本地远程分支)

---

## 创建远程仓库

`git clone --bare thirdPartyJniSo git@gitee.com:chenjim/thirdPartyJniSo.git`  用带版本的项目创建纯版本仓库  
`scp -r thirdPartyJniSo git@git.csdn.net:~`  将纯仓库上传到服务器上  
`mkdir robbin_site.git && cd robbin_site.git && git --bare init`  在服务器创建纯仓库  
`git remote add origin git@gitee.com:chenjim/thirdPartyJniSo.git`  设置远程仓库地址  
`git push -u origin master`  客户端首次提交  
`git push -u origin develop`  首次将本地develop分支提交到远程develop分支，并且track  
`git remote set-head origin master`  设置远程仓库的HEAD指向master分支

---

## tag 使用

创建带有说明的标签，用-a指定标签名，-m指定说明文字：  
`git tag -a VV1.0 -m "version 1.0 released push url" d5a65e9`  
`git show V1.0`  查看标签V1.0信息  
`git tag V1.0`   给当前commit添加tag V1.0  
`git tag V1.0 471fd27` 给指定commit 471fd27 添加tag  
`git push origin V1.0` 将指定tag推送到远程  
`git push --tags`        将所有tag推送到远程  
`git push origin --tags` 将所有tag推送到远程  
`git tag -d V1.0`   删除本地tag  
`git push origin :V1.0`             删除远程tag  
`git push origin --delete tag V1.0` 删除远程tag  
`git ls-remote --tags origin`  查询远程tags

[Git 如何同步本地tag与远程tag](https://blog.csdn.net/wei371522/article/details/83186077)

- 问题场景：  
  同事A在本地创建tagA并push同步到了远程   
  ->同事B在本地拉取了远程tagA(git fetch)   
  ->同事A工作需要将远程标签tagA删除  
  ->同事B用git fetch同步远端信息，git tag后发现本地仍然记录有tagA

- 分析：  
  对于远程repository中已经删除了的tag，  
  即使使用git fetch --prune，甚至"git fetch --tags"确保下载所有tags，也不会让其在本地也将其删除的。  
  而且，似乎git目前也没有提供一个直接的命令和参数选项可以删除本地的在远程已经不存在的tag

- 解决方法：  
  `git tag -l | xargs git tag -d`   #删除所有本地tag分支    
  `git fetch origin --prune`  #从远程拉取所有信息

---

## reflog 使用

> reflog是Git操作的一道安全保障，它能够记录几乎所有本地仓库的改变。  
> 包括所有分支commit提交，已经删除（其实并未被实际删除）commit都会被记录。  
> 总结：只要HEAD发生变化，就可以通过reflog查看到。

`git reflog`               查看最近的git变化  
`git checkout HEAD@{90}`   将指定的变化迁出


---


**相关链接**

- [安卓软件开发常用命令集合](https://h89.cn/archives/180.html)

