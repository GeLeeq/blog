---
layout: post
title:  初识hadoop
date:   2016-04-11 17:15:00
category: "Hadoop"
---
**谷歌三大论文：GFS、 MapReduce、 BigTable  (2003、2004、2006)**

- HDFS实现数据的存储  
- MapReduce实现数据的分析和处理（将硬盘的读写问题转换为对一个数据集(key-value)的计算）

**MapReduce对比传统技术（关系数据库、网络计算）：**  
1. 大量硬盘寻址的时间  `>>>` 流数据读取模式，读取全部数据（取决于硬盘传输速率）  
2. 基于消息传递接口的数据处理时间（网络传输数据）`>>>` 数据尽量在本地，快速访问处理（数据本地化）  
对于`2`的补充：  
大规模分布式计算环境下，协调各进程的执行（是否挂了，挂了能继续完成整个计算）。MapReduce更有优势，
能够检测并重新执行失败的任务，无共享框架（任务彼此独立）。传统的基于MPI（消息传递接口）必须显式管理检查点和恢复机制。

## Hadoop发行版本： ##
`1.x`版本(0.22之前）     `0.22`发行版本     `2.x`版本(0.23及之后, *YARN、HDFS联邦管理与高可用*)

兼容性：  
主发行版本允许破坏API兼容性(1.x.y 到 2.0.0)  
次重点发行版本和单点发行版本不应该破坏兼容（`@InterfaceStability.Stable、@InterfaceStability.Evolving`）

## MapReduce简介： ##
MapReduce任务过程分为两个处理阶段：map阶段和reduce阶段。  
如下图：  
![mapreduce过程图](http://geleeq.github.io/blog/post_res/images/hadoop/mapreduce-1.jpg)  


Hadoop有三种运行模式：`独立（本地）模式、伪分布模式、全分布模式`  

	hadoop默认的配置是独立模式（用户须配置JAVA_HOME，见conf/hadoop-env.sh)，
	非分布式的，仅当作java进程，非常适用于debug。 其他模式的配置见官网。

例子：  

>官方例子：

	$ mkdir input
	$ cp etc/hadoop/*.xml input
	$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar grep input output 'dfs[a-z.]+'
	$ cat output/*

>书本例子：

	$ export HADOOP_CLASSPATH=hadoop-examples.jar
	$ hadoop MaxTemperature input/ncdc/sample.txt output

第二个运行的是具体的类，需要定义一个HADOOP_CLASSPATH环境变量用于添加应用程序类的路径。

>TIPS: reducer输出目录不应该已经存在的，否则hadoop会报错并拒绝运行作业`Job`

过程输出中可以查看到的主要信息：

	1. 任务标识 job_local_0001
	2. map、reduce作业标识 attempt_local_0001_m_000000_0  attempt_local_0001_r_000000_0
	3. 作业统计信息，如：map及reduce输入输出数量
每个reducer都有一个输出文件名为 `part-r-nnnn`。(其中nnnn是从0000开始的分块序号）

### 新旧MapReduce API比较 ###

- 旧API使用`Mapper`和`Reducer`接口，新API中使用虚类，更好扩展
- 新API使用上下文对象与MapReduce系统通信，`Context`统一了旧API中的`JobConf`、`OutputCollector`  
  和`Reporter`的功能
- 包名，`org.apache.hadoop.mapreduce`新、`org.apache.hadoop.mapred`旧
- 新API通过重写run()方法允许mapper和reducer控制执行流程，而旧API只能通过写`MapRunnable`类来实现，  
  且reducer没有对等的实现
- 新API作业控制由`Job`类实现，旧API使用`JobClient`类
- 新API配置由`Configuration`来完成，旧API使用`JobConf`
- 输出命名part-nnmm变为part-m-nnnn/part-r-nnnn

### 横向扩展（使用HDFS)