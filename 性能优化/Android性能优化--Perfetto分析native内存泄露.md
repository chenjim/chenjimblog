
# Android性能优化--Perfetto分析native内存泄露


>本地首发地址 <https://blog.csdn.net/CSqingchen/article/details/128382445>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>  
>官方文档(可在Chome直接翻译)  <https://perfetto.dev/docs/data-sources/native-heap-profiler>  
>示例 raw-trace 资源地址 <https://download.csdn.net/download/CSqingchen/87321798>  
>本文示例是windows，这里使用了python工具，在Linux和mac同样适用  

1. 首先安装python3环境，参见 <https://www.python.org/downloads/>
2. 下载 perfetto ，地址在 <https://github.com/google/perfetto>  
  后面需要用到这里的 `perfetto\tools\heap_profile`  本文放在了目录 `D:\tools\perfetto`
3. 抓取一次某个应用的内存命令如下，注意提前关闭其它adb程序，如AS   
`python D:\tools\perfetto\tools\heap_profile -n com.app.package.name`    
![](https://pic.chenjim.com/202304171210123.png-blog)
这里只能抓到一次内存的快照，如果想连续记录多次内存的数据需要能Root手机  
**首次运行**会有如下联网下载  
`Downloading https://commondatastorage.googleapis.com/perfetto-luci-artifacts/v32.1/windows-amd64/traceconv.exe`  
如果下载失败，Windows系统请 [从连接下载](https://download.csdn.net/download/CSqingchen/87321798) 并参考里面说明放文件。  
或者**参考**如下，修改`tools/heap_profile` 中 curl 下载方式
	```shell
	diff --git a/tools/heap_profile b/tools/heap_profile
	@@ -243,8 +243,9 @@ def download_or_get_cached(file_name, url, sha256):
	   if needs_download:
	     # Either the filed doesn't exist or the SHA256 doesn't match.
	     tmp_path = bin_path + '.tmp'
	+    proxy_7890='http://127.0.0.1:7890'
	     print('Downloading ' + url)
	-    subprocess.check_call(['curl', '-f', '-L', '-#', '-o', tmp_path, url])
	+    subprocess.check_call(['curl', '-f', '-L', '-#', '-x', proxy_7890, '-o', tmp_path, url])
	     with open(tmp_path, 'rb') as fd:
	       actual_sha256 = hashlib.sha256(fd.read()).hexdigest()
	     if actual_sha256 != sha256:
	```

4. 连续抓取多次内存快照  
	`adb shell killall -USR1 heapprofd`    需要Root权限，上一步骤不要停止  
	每执行一次，上一步会记录一次  
	这里我上传了一份自己抓的数据,下载地址 <https://download.csdn.net/download/CSqingchen/87321798>   
5. 使用 `perfetto` 分析抓到的 `raw-trace` 文件，即从 <https://ui.perfetto.dev/> 打开 `raw-trace` 文件  
    ![](https://pic.chenjim.com/202304171212221.png-blog)
    通过点击方块，对比不用时刻的内存。  
    可以看到第一个大块有内存一直上升，结合其中的栈堆，分析并解决即可。  
    有时很小泄露，不容易看出，可以反复很多次操作应用后，对比前后数据  
    下载资源 `raw-trace.02` 是解决问题后，抓取的tarce，可以看到问题已解决  

> 相关问题思考  
- [AndroidStudio Profile](https://developer.android.com/studio/profile/android-profiler) 也可以Dump内存，但内存分析没有这个直接
- [LeakCanary](https://square.github.io/leakcanary/) 也可以分析内存，主要是用来分析View视图，没法分析这个native内存的数据

>相关文章  
[Android性能优化--Perfetto抓取trace](https://blog.csdn.net/CSqingchen/article/details/128900541)  
[Android性能优化--perfetto分析native内存泄露](https://blog.csdn.net/CSqingchen/article/details/128382445)  
[Android性能优化--Perfetto用SQL性能分析](https://blog.csdn.net/CSqingchen/article/details/134167741)  
