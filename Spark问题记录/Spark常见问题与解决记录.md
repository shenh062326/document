## Spark常见问题记录

#### Block丢失问题
* 现象

```
SparkStreaming任务经常出现block丢失导致任务失败，从日志看到receiver接收的block在处理前就被删除。
17/09/20 14:10:04 INFO scheduler.JobScheduler: Finished job streaming job 1505887650000 ms.0 from job set of time 1505887650000 ms
		1505887650000 2017/9/20 14:7:30
		1505887650000
17/09/20 14:10:04 INFO scheduler.JobScheduler: Total delay: 154.896 s for time 1505887650000 ms (execution: 154.722 s)
17/09/20 14:10:04 DEBUG scheduler.JobGenerator: Got event ClearMetadata(1505887650000 ms)
17/09/20 14:10:04 DEBUG streaming.DStreamGraph: Clearing metadata for time 1505887650000 ms
17/09/20 14:10:04 DEBUG dstream.ForEachDStream: Clearing references to old RDDs: []
17/09/20 14:10:04 DEBUG dstream.ForEachDStream: Unpersisting old RDDs:
17/09/20 14:10:04 DEBUG dstream.ForEachDStream: Cleared 0 RDDs that were older than 1505887605000 ms:

17/09/20 14:10:05 INFO storage.BlockManagerInfo: Removed input-24-1505887598000 on antspark-common-49-17.et15.alipay.com:49575 in memory (size: 17.7 MB, free: 21.6 GB)


input-24-1505887598000    2017/9/20 14:6:38


17/09/20 14:10:06 INFO scheduler.JobScheduler: Finished job streaming job 1505887605000 ms.0 from job set of time 1505887605000 ms
17/09/20 14:10:06 INFO scheduler.JobScheduler: Total delay: 201.305 s for time 1505887605000 ms (execution: 201.173 s)
17/09/20 14:10:06 DEBUG scheduler.JobGenerator: Got event ClearMetadata(1505887605000 ms)
17/09/20 14:10:06 DEBUG streaming.DStreamGraph: Clearing metadata for time 1505887605000 ms
```

* 分析
	* spark streaming现在的删除机制：
当一个batch结束后，会触发删除比这个batch老的数据，默认情况下spark.streaming.concurrentJob=1，顺序执行，删除老的数据是没问题的。
但是这个任务的并行度配置为spark.streaming.concurrentJob=6，就可能出现时间晚的batch结束，触发删除此前的数据，而老的数据还没处理或正在处理，这时候就出错。

* 解决方法：有三种
	* spark.streaming.concurrentJobs  1
	* spark.streaming.unpersist=false   Block只会在内存不足时触发删除，容易触发OOM。
	* Block保存较长时间，但不能彻底解决 dstream().remember(Durations.seconds(batchTime * 10));

#### SparkSQL中文乱码问题
* 现象

```
通过SparkSQL向表中插入中文，然后读出，显示为乱码
```

* 解决方法

```
ON Yarn模式，需要设置Executor的环境变量
spark.executorEnv.LANG=en_US.UTF-8
spark.yarn.appMasterEnv.LANG=en_US.UTF-8

Client模式，设置spark-env.sh中的环境变量
export LANG=en_US.UTF-8
```

#### 问题
