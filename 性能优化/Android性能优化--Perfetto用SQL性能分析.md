

[TOC]

>本文首发地址 <https://blog.csdn.net/CSqingchen/article/details/134167741>  
>最新更新地址 <https://gitee.com/chenjim/chenjimblog>  
>Perfetto 抓取 trace 可参考 <https://blog.csdn.net/CSqingchen/article/details/128900541>

## 介绍
Perfetto 是一个由 Google 开发的高性能、可扩展的事件追踪系统，用于在实时和离线场景下监控系统的性能。  
它通过一种简单且强大的查询语言（称为 SQL）来分析和查询事件数据。  
在本博客中，我们将深入探讨如何使用 SQL 在 Perfetto 中进行性能分析。

## Perfetto SQL 基础  
Perfetto SQL 是一种用于查询事件数据的语言，它支持大多数标准的 SQL 操作，  
如 SELECT、FROM、WHERE、GROUP BY、ORDER BY 等。  
在 Perfetto 中，数据以表格的形式存储，因此你可以使用 SQL 来检索和操作这些数据。  
下面是一个简单的 Perfetto SQL 查询示例：  
`SELECT ts, dur, name FROM slice WHERE ts > 85545835986081 AND ts < 85546017415330`  
`ts, dur, name` 是挑选的字段，`slice` 是挑选的表名  
![](https://pic.chenjim.com/202311031127354.png-blog)   
示例 trace 文件可以在 [data/perfetto](https://gitee.com/chenjim/chenjimblog/blob/master/data/perfetto)下载 xiaomi13.camera.trace.7z   
解压后，在 https://ui.perfetto.dev/ 打开  

可以使用如下命令查看表中有哪些字段   
`SELECT * FROM slice LIMIT 10`   

如果 trace 中包含 android log，还可以用如下命令过滤日志  
`SELECT * FROM android_logs  WHERE msg LIKE "%ProcessRequest%" LIMIT 30`

trace 中有哪些表可用，以及各个字段是什么关系呢，可以参考  
<https://perfetto.dev/docs/analysis/sql-tables>  
其中 Event 关系图如下 
![](https://pic.chenjim.com/202311031631881.png-blog)  


## 使用 Perfetto SQL 进行性能分析
使用 Perfetto SQL 进行性能分析的关键在于理解如何构造查询以获取你需要的信息。  
以下是一些常见的性能分析任务和相应的 SQL 查询示例：  
1. 分析特定事件的发生频率：  
   `SELECT COUNT(*) FROM slice WHERE name = 'waitForNextFrame'`  
   `waitForNextFrame` 一共有多少次   
2. 分析事件的性能数据：  
   `SELECT (dur/1e6) FROM slice WHERE name = 'waitForNextFrame'`  
   每次 `waitForNextFrame` 耗时多少毫秒。`dur`单位是纳秒
3. 分析一段时间内的事件数据：
   `SELECT MIN(dur/1e6) as min_duration, MAX(dur/1e6) as max_duration, AVG(dur/1e6) as avg_duration FROM slice WHERE name = 'waitForNextFrame' and dur > 0`
   显示 `waitForNextFrame` 最小、最大、平均值
4. 对事件进行排序：  
   `SELECT (dur/1e6),ts,name FROM slice WHERE name LIKE '%wait%' and dur > 0 ORDER by dur DESC`  
5. 统计 CPU 时间  
   ```sql
    DROP VIEW IF EXISTS slice_with_utid;
    CREATE VIEW slice_with_utid AS
    SELECT
    ts,
    dur,
    slice.name as slice_name,
    slice.id as slice_id, utid,
    thread.name as thread_name
    FROM slice
    JOIN thread_track ON thread_track.id = slice.track_id
    JOIN thread USING (utid);

    DROP TABLE IF EXISTS slice_thread_state_breakdown;
    CREATE VIRTUAL TABLE slice_thread_state_breakdown
    USING SPAN_LEFT_JOIN(
    slice_with_utid PARTITIONED utid,
    thread_state PARTITIONED utid
    );

    SELECT slice_id, slice_name, SUM(dur) AS cpu_time
    FROM slice_thread_state_breakdown
    WHERE state = 'Running'
    GROUP BY slice_id;
   ```

基本都是 SQL 语句，SQL关键字含义可以参考 <https://www.w3schools.cn/sql/>

## 总结
使用 Perfetto 和 SQL 进行性能分析是一种强大而灵活的方法。
通过理解如何构造 SQL 查询，你可以轻松地获取你需要的信息，从而更好地理解系统的性能。
在 Perfetto 中使用 SQL 进行性能分析可以帮助你更好地理解系统的性能，并找出潜在的性能问题。


相关文章  
[Android性能优化--Perfetto抓取trace](https://blog.csdn.net/CSqingchen/article/details/128900541)  
[Android性能优化--perfetto分析native内存泄露](https://blog.csdn.net/CSqingchen/article/details/128382445)  
[Android性能优化--Perfetto用SQL性能分析](https://blog.csdn.net/CSqingchen/article/details/134167741)


参考文章  
<https://perfetto.dev/docs/quickstart/trace-analysis>  
<https://perfetto.dev/docs/analysis/common-queries>  
<https://zhuanlan.zhihu.com/p/641412977>  
<https://yiyan.baidu.com/share/gdFw3P5ucI>  