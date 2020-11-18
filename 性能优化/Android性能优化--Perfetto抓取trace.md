

>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/128900541>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>  
>Perfetto 官方链接地址 <https://github.com/google/perfetto/>  

## 介绍
Perfetto 是基于 Android 的系统追踪服务， Android的trace跟踪服务在 Android11(R) 之后是默认打开的，但是如果你是 Android 9 ( P ) 或者 10 ( Q ) ，那么就需要手动设置一下相应的 prop 属性。  
`adb shell setprop persist.traced.enable 1`

---

## 使用 adb 抓取  
`adb shell perfetto -o /data/misc/perfetto-traces/trace_file.perfetto-trace -t 60s sched freq idle am wm gfx view binder_driver hal dalvik camera input res memory`  
`-o /data/misc/perfetto-traces/trace_file.perfetto-trace` 输出trace的路径  
`-t 60s` 最多抓取时长，可以Ctrl+C停止  
`sched ... memory` 要抓取相关模块
更多配置说明，参见文档：<https://perfetto.dev/docs/concepts/config>
使用配置文件，通过adb抓取，参考如下  
`adb push atrace.cfg /data/local/tmp/atrace.cfg`  
`adb shell "cat /data/local/tmp/atrace.cfg | perfetto --txt -c - -o /data/misc/perfetto-traces/trace.perfetto-trace"`  
atrace.cfg 是配置信息，默认最多只抓取10秒，可以修改更长  
可以Ctrl+C停止，需执行`adb shell`  
再执行 `cat /data/local/tmp/atrace.cfg | perfetto --txt -c - -o /data/misc/perfetto-traces/trace.perfetto-trace`  
更多配置参见 <https://github.com/google/perfetto/blob/master/test/configs>  
然后 `adb pull /data/misc/perfetto-traces/trace.perfetto-trace` ，在 <https://perfetto.dev/> 导入文件分析  

---

## 通过 perfetto 网页抓取  
打开 <https://ui.perfetto.dev/#!/record>  
"Add ADB Device" 选择手机设备  
如下图，有一些参数配置，根据自己的需要添加修改     
![](https://pic.chenjim.com/202304171253853.png-blog)
最后右上角 "Start Recording"  

---

## 直接在手机上抓取  
开启开发者模式：系统设置--关于手机--连续点击。部分机型可网上搜索开启方式。  
进入开发者模式，选择"系统跟踪"，里面有一些相关设置类别，可以选择要抓取的数据种类  
抓取的trace信息会直接存储在手机 `/data/local/traces`，"查看跟踪文件"，可以分享  
此方式比较适合没有电脑，随时抓取

---

## 使用 record_android_trace 抓取  
执行如下命令，会自动抓取，并打开 <https://perfetto.dev/> 分析  
python [record_android_trace](https://github.com/google/perfetto/blob/master/tools/record_android_trace) -c [atrace.cfg](https://github.com/google/perfetto/blob/master/test/configs/atrace.cfg) -o out.perfetto.trace  
record_android_trace 是 perfetto 中文件，具体参见以上命令的链接  
atrace.cfg 是配置信息，更多配置说明参见文档：<https://perfetto.dev/docs/concepts/config>  
默认只抓取10秒，可以修改更长，可以Ctrl+C停止，更多配置参见 <https://github.com/google/perfetto/blob/master/test/configs>  
首次运行会从 googleapis.com 下载文件，如果遇到超时，可能需要修改如下  
Windows可参考 [ Android基于perfetto分析native内存泄露](https://editor.csdn.net/md/?articleId=128382445) 放文件

```shell
diff --git a/tools/heap_profile b/tools/heap_profile
@@ -243,8 +243,9 @@ def download_or_get_cached(file_name, url, sha256):
	if needs_download:
		# Either the filed doesn't exist or the SHA256 doesn't match.
		tmp_path = bin_path + '.tmp'
+    proxy_7890='http://127.0.0.1:7890'
		print('Downloading ' + url)
-    subprocess.check_call(['curl', '-f', '-L', '-#', '-o', tmp_path, url])
+    subprocess.check_call(['curl', '-f', '-L', '-#','-x',proxy_7890, '-o', tmp_path, url])
		with open(tmp_path, 'rb') as fd:
		actual_sha256 = hashlib.sha256(fd.read()).hexdigest()
		if actual_sha256 != sha256:
```

---

## 熟悉 perfetto 快捷键，会有事半功倍效果
![](https://pic.chenjim.com/202304171254826.png-blog)

----

## 注意事项
- 最好先执行`adb shell`，然后执行` perfetto `相关命令，因为在某些情况 Ctrl+C不通过ADB传播而无法停止.
- 在 Android 12 之前的非 root 设备上，由于过度限制的 SELinux 规则，配置只能通过 (cat config | adb shell perfetto -c -) 传递 。由于 Android 12 `/data/misc/perfetto-configs`可用于存储配置。
- 在 Android 10 之前的设备上，adb 无法直接拉取 `/data/misc/perfetto-traces`. 可以用 `adb shell cat /data/misc/perfetto-traces/trace > trace`变通。
- 当捕获较长的跟踪时，例如在基准测试或 CI 的上下文中，使用 `PID=$(perfetto --background)`然后`kill $PID`停止。

---


----

- 一个参考配置示例如下  
  更多可参见  <https://github.com/google/perfetto/blob/master/test/configs>
	```
	buffers: {
	    size_kb: 707200
	    fill_policy: RING_BUFFER
	}
	buffers: {
	    size_kb: 707200
	    fill_policy: RING_BUFFER
	}
	data_sources: {
	    config {
	        name: "linux.process_stats"
	        target_buffer: 1
	        process_stats_config {
	            scan_all_processes_on_start: true
	            proc_stats_poll_ms: 1000
	        }
	    }
	}
	data_sources: {
	    config {
	        name: "android.log"
	        android_log_config {
	        }
	    }
	}
	data_sources: {
	    config {
	        name: "android.surfaceflinger.frametimeline"
	    }
	}
	data_sources: {
	    config {
	        name: "linux.sys_stats"
	        sys_stats_config {
	            meminfo_period_ms: 1000
	            vmstat_period_ms: 1000
	            stat_period_ms: 1000
	            stat_counters: STAT_CPU_TIMES
	            stat_counters: STAT_FORK_COUNT
	        }
	    }
	}
	data_sources: {
	    config {
	        name: "android.heapprofd"
	        target_buffer: 0
	        heapprofd_config {
	            sampling_interval_bytes: 4096
	            shmem_size_bytes: 8388608
	            block_client: true
	        }
	    }
	}
	data_sources: {
	    config {
	        name: "android.java_hprof"
	        target_buffer: 0
	        java_hprof_config {
	        }
	    }
	}
	data_sources: {
	    config {
	        name: "linux.ftrace"
	        ftrace_config {
	            ftrace_events: "sched/sched_switch"
	            ftrace_events: "power/suspend_resume"
	            ftrace_events: "sched/sched_wakeup"
	            ftrace_events: "sched/sched_wakeup_new"
	            ftrace_events: "sched/sched_waking"
	            ftrace_events: "power/cpu_frequency"
	            ftrace_events: "power/cpu_idle"
	            ftrace_events: "sched/sched_process_exit"
	            ftrace_events: "sched/sched_process_free"
	            ftrace_events: "task/task_newtask"
	            ftrace_events: "task/task_rename"
	            ftrace_events: "lowmemorykiller/lowmemory_kill"
	            ftrace_events: "oom/oom_score_adj_update"
	            ftrace_events: "ftrace/print"
	            ftrace_events: "binder/*"
	            atrace_categories: "input"
	            atrace_categories: "gfx"
	            atrace_categories: "view"
	            atrace_categories: "webview"
	            atrace_categories: "camera"
	            atrace_categories: "dalvik"
	            atrace_categories: "power"
	            atrace_categories: "wm"
	            atrace_categories: "am"
	            atrace_categories: "ss"
	            atrace_categories: "sched"
	            atrace_categories: "freq"
	            atrace_categories: "binder_driver"
	            atrace_categories: "aidl"
	            atrace_categories: "binder_lock"
	            atrace_apps: "*"
	        }
	    }
	}
	duration_ms: 300000
	flush_period_ms: 30000
	incremental_state_config {
	    clear_period_ms: 5000
	}
	write_into_file: true
	```
----

**相关文章连接**  
[Android性能优化--Perfetto抓取trace](https://blog.csdn.net/CSqingchen/article/details/128900541)  
[Android性能优化--perfetto分析native内存泄露](https://blog.csdn.net/CSqingchen/article/details/128382445)  
[Android性能优化--Perfetto用SQL性能分析](https://blog.csdn.net/CSqingchen/article/details/134167741)

