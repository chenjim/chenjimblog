
[TOC]

> 本文收发地址 <https://h89.cn/archives/130.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>   
> Android Studio 是安卓开发的最强工具，本文主要介绍一些个人配置，以提高我们的开发效率


## Markdown Not Support JCEF
1. 问题  
Your environment does not support JCEF,can not use Mardkdown Editor Preview  
Android Studio 默认运行时暂不支持JCEF，无法使用markdown预览  
2. 解决办法  
  方法1：默认运行时路径在 `android-studio\jbr` , 可以备份后，替换为带`JCEF`的运行时  
  方法2：安装新的运行时： `Ctrl + Shift + A`，选择 `Actions` , 搜索 `runtime`  
  选择 `Choose Boot Java Runtime for the IDE`, 选择包含`JCEF`的 Runtime  
  确认后，会自动下载 `jbr`，默认保存在目录 `~/.jbr/`  
  ![](https://pic.chenjim.com/202308021552566.png-blog)  
  当 AS 更新后，可能需要下载新的 `jbr` 适配  

---

## Version Control 中 Local Changes 不显示  
   Settings --> Version Control --> Commit --> Use non-modal commit interface  
   取消如上勾选   
   ![](https://pic.chenjim.com/202401242227241.png-blog)


---

## 自动同步不同电脑 Android Studio 配置
1. 背景  
  通过 `settings repository` 插件，可以将 IDEA 的设置(如快捷键等)同步到 Git 仓库，实现在不同电脑使用相同的设置  
  ![](https://pic.chenjim.com/202308021546764.png-blog)   
  ![](https://pic.chenjim.com/202308021547639.png-blog)  

2. 问题  
  更新 Android Studio 到  2022.3.1，移除了插件 `Settings Repository`   
  导致无法正常使用 `Manage IDE Settings`, 自动同步 IDEA 的配置  
  jetbrains 插件中 [已经废弃 Settings Repository](https://plugins.jetbrains.com/plugin/7566)  


3. 解决办法  
  1. 禁用(卸载) `Settings Sync` 
  2. 重新安装插件 `Settings Repository`
  3. 重启 Studio  
   
4. 问题后续 
  [Android Studio](https://developer.android.com/studio/) 基于 [IntelliJ IDEA](https://www.jetbrains.com/idea/)开发的，使用 `Settings Sync` 是趋势  
  但是 2022.3.1 、2023.1.1 、2023.2.1 预览版无法使用 `Settings Sync`  
  原因是 `com/jetbrains/cloudconfig/` 中部分类未打包到 Studio , Studio 登录的是 Google 账户，IDEA 是 jetbrains 账户    
  可参考链接 <https://youtrack.jetbrains.com/issue/IDEA-322921>  
  预计 Studio 2023.3 会支持 `Settings Sync`

---

## 自动格式化代码 
`Action on Save`--> `Reformat code` 注意选择 `Changed lines`, 参考如下图配置  
![](https://pic.chenjim.com/202401242220471.png-blog)

## 自动导入包  
`Auto Import` -->勾选 `Add unambiguous imports on the fly` 参考如下图配置  
![](https://pic.chenjim.com/202401242224662.png-blog)

## 一些好用插件  
- Alibaba Java Coding Guidelines  
  代码格式异常提示  
- Android Drawable Preview  
  预览 Drawable xml 文件  
- Rainbow Brackets  
  彩虹括号 
- MultiHighlight  
  高亮代码 
- Translation  
  翻译插件 
