---
layout: post
title:  Hadoop分布式文件系统
date:   2016-04-13 10:53:00
category: "Hadoop"
---

管理网络中跨多台计算机存储的文件系统秒为分布式文件系统（`distributed filesystem`)
 
### HDFS特点　###

- 能存储超大文件
- 流式数据访问（一次写入，多次读取）
- 节点故障能够继续运行
- 低时间延迟访问（几十毫秒）不适合在HDFS上运行（使用HBase）
- namenode将文件系统的元数据存储在内存中，存储的文件总数受内存限制
- 不支持多用户写入，不支持任意位置修改文件（仅文件末尾添加）

### HDFS基本概念 ###
每个磁盘都有默认的数据块大小（一般为512字节），这是磁盘数据读写的最小单位。文件系统块的大小可以是磁盘块的整数倍，HDFS的数据块默认是64MB，HDFS与其他文件系统不同的是，小于一个块大小的文件不会占据整个块的空间。

HDFS块设计成这么大，是为了最小化寻址开销（传输数据时间明显大于定位块的时间）；但也不能设置过大，因为MapReduce中的map任务通常一次只处理一个块中的数据，所以如果任务数太少（少于集群节点数量），作业运行速度就会比较慢。

HDFS数据块的好处：

- 易维护（且文件元数据不存储在块中，而是内存上)
- 高可用性（块可以复制多份在其他节点上）

namenode和datanode:

>HDFS集群中有两类节点以管理者-工作者模式运行： 一个namenode，多个datanode

- namenode维护着文件系统树及整棵树内所有的文件和目录，永久保存在磁盘上：`命名空间镜像文件`和`编辑日志文件`

- namenode也记录文件的块数据节点信息，但它并不永久保存，因为这些信息会在系统启动时由datanode重建

- datanode是文件系统的工作节点，它们根据需要存储并检索数据块（受client和namenode调度），并定期向namenode发送它们所存储的块的列表。

- client代表用户通过与namenode和datanode交互来访问整个文件系统，提供类似于`POSIX`（可移植操作系统接口）

- namenode容错方案1：备份namenode上保存文件系统元数据的持久状态文件；这些写操作是实时同步，原子性操作，写入本地磁盘的同时，写入一个远程挂载的`网络文件系统`（`NFS`）。
- namenode容错方案2：运行一个辅助namenode，定期通过编辑日志合并命名空间镜像。所以总是滞后于主节点，当主节点全部失效时，将`NFS`上的namenode元数据复制到辅助namenode并作为新的主namenode运行。

- 集群横向扩展（2.x版本引入`联邦HDFS`）：namenode在内存中保存的文件与块的映射关系，当拥有大量文件时，内存成为瓶颈；2.x的Hadoop版本允许通过添加namenode实现扩展，每个namenode管理文件系统命名空间的一部分（如：/user、/share），该功能客户端通过`ViewFileSystem`和viewfs: //URI进行配置和管理。

### 命令行接口 ###

	$ bin/hadoop fs -mkdir /tmp
	$ bin/hadoop fs -copyFromLocal /data/txt/example.txt hdfs://localhost/tmp/example.copy.txt
	$ bin/hadoop fs -copyFromLocal /data/txt/example.txt /example.copy.txt （省略URI)
	$ bin/hadoop fs -ls /tmp

>Hadoop文件系统是一个抽象概念，由java抽象类org.apache.hadoop.fs.FileSystem定义，HDFS就是它的一个具体实现(hdfs.DistributedFileSystem)。类似还有其他实现，如：fs.LocalFileSystem、hdfs.HftpFileSystem(在HTTP上提供对HDFS只读访问的文件系统)、fs.s3native.NativeS3FileSystem(由Amazon S3支持的文件系统)。  

>尽管MapReduce程序可以访问任何文件系统，但处理大数据集时，使用有数据本地优化的分布式文件系统，如HDFS。

### 附录 ###
伪分布式配置：  

>配置hadoop的默认文件系统，URI指定，localhost上默认8020端口 

![](http://geleeq.github.io/blog/post_res/images/hadoop/hadoop-config.jpg)