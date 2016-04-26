---
layout: post
title:  Hadoop分布式文件系统
date:   2016-04-13 10:53:00
category: "Hadoop"
---

管理网络中跨多台计算机存储的文件系统称为分布式文件系统（`distributed filesystem`)
 
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

以上配置是老版本的，修正：
	
	--  core-site.xml
    hadoop.tmp.dir的配置不是在hdfs-site.xml上了	// 默认值/tmp/hadoop-${user.name}
	--  mapred-site.xml
	<name>mapreduce.framework.name</name>
	<value>yarn</value> //默认local，使用YARN时需要设置成yarn
	--  yarn-site.xml
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>


### 接口 ###
Hadoop是用java写的，所以通过Java API可以调用所有Hadoop文件系统的交互操作。

- HTTP
	- 内嵌的web服务器 `WebHDFS` 目录服务namenode 50070、数据流传输datanode 50075
	- 独立代理服务器 `HttpFS`
- C语言
	- `libhdfs`的C语言库，基于JNI
	- 与Java API相似，滞后
- FUSE
	
	介绍：用户空间文件系统(Filesystem in Userpace, FUSE), 允许把按照用户空间实现的文件系统整合成一个Unix文件系统。
	
	- 通过使用Hadoop的`Fuse-DFS`功能模块，任何Hadoop文件系统(一般不为hdfs)均可以作为一个标准文件系统进行挂载。
	- `Fuse-DFS`是用C语言实现的 `src/contib/fuse-dfs`

### Java接口 ###
- 通过java.net.URL完成简单交互， 但需要setURLStreamHandlerFactory，且JVM只允许调用一次该方法。
		
		static{
			URL.setURLStreamHandlerFactory(
				new org.apache.hadoop.fs.FsUrlStreamHandlerFactory());		
		}
		InputStream in = null;
		try{
			in = new URL("hdfs://host/path").openStream();
			org.apache.hadoop.io.IOUtils.copyBytes(in, System.out, 4096, false);
		} finally {
     	 	org.apache.hadoop.io.IOUtils.closeStream(in);
    	}
- 使用FileSystem API
		
		//读取数据
		import org.apache.hadoop.conf.Configuration;
		import org.apache.hadoop.fs.FileSystem;
		import org.apache.hadoop.fs.Path;
		import org.apache.hadoop.io.IOUtils;
		
		public class FileSystemCat {
		  public static void main(String[] args) throws Exception {
		    String uri = args[0];
		    Configuration conf = new Configuration();
		    FileSystem fs = FileSystem.get(URI.create(uri), conf);//多个重载
		    InputStream in = null;
		    try {
		      in = fs.open(new Path(uri));//很关键， Hadoop Path对象来代表文件
		      IOUtils.copyBytes(in, System.out, 4096, false);
		    } finally {
		      IOUtils.closeStream(in);
		    }
		  }
		}
	
		org.apache.hadoop.fs.FSDataInputStream in = fs.open(new Path(uri));//open实际返回的对象
		FSDataInputStream extends DataInputStream implements Seekable, PositionedReadable{}
		in.seek(0); //重新定位到文件中任意一个绝对位置，超出会引发IOException; 高开销操作，建设使用MapReduce代替
		in.read(long position, byte[] buffer, int offset, int length);//指定偏移量处读取文件一部分；高开销操作
		//写入数据
        public class FileCopyWithProgress {
		  public static void main(String[] args) throws Exception {
		    String localSrc = args[0];
		    String dst = args[1];
		    
		    InputStream in = new BufferedInputStream(new FileInputStream(localSrc));
		    
		    Configuration conf = new Configuration();
		    FileSystem fs = FileSystem.get(URI.create(dst), conf);
			//很关键： 能够自已创建目录、Progressable回调接口，写入数据进度
			//还有一种方式使用append()方法在已有文件末尾追加数据
		    OutputStream out = fs.create(new Path(dst), new Progressable() {
		      public void progress() {
		        System.out.print(".");
		      }
		    });
		    
		    IOUtils.copyBytes(in, out, 4096, true);
		  }
		}
		
		FSDataOutputStream out =  fs.create(new Path(dst)); //实际返回的对象
		有个查询文件当前位置的方法 long getPos()；
		没有在指定位置进行写入的功能，hadoop仅允许在文件末尾位置写入。
		
		//创建目录
        public boolean mkdirs(Path f) throws IOException
        
        //文件/目录是否存在
        public boolean exists(Path f) throws IOException
		
		//文件或目录元数据
		//FileStatus 长度、块大小、修改时间、所有者、....
        Path file = new Path("/dir/file");
    	FileStatus stat = fs.getFileStatus(file);
		//列出文件和目录
        fs.listStatus(Path f);
		fs.listStatus(Path f, PathFilter filter);
		fs.listStatus(Path[] files);
		fs.listStatus(Path[] files, PathFilter filter);
        
        返回FileStatus[]对象，如果f是文件，则返回长度为1的数组
		PathFilter接口boolean accept(Path path)限制匹配文件/目录
		FileUtil的stat2Paths(status)方法，将FileStatus[] 转换为 Path[]

		//列出通配的目录(如: /*)中的文件
		public FileStatus[] globStatus(Path pathPattern) throws IOException
		public FileStatus[] globStatus(Path pathPattern, PathFilter filter) throws IOException

		//删除文件或目录
		public boolean delete(Path f, boolean recursive) throws IOException
        只有在recursive为true时，非空目录及其内容才会被删除

### 数据流 ###

- 文件读取 （FSDataInputStream、DFSInputStream、网络拓扑）
- 文件写入 （FSDataOutputStream、DFSOutputStream、DataStreamer处理数据队列、故障确认机制、）
- 一致模型 （hflush()、hsync()）
- distcp复制 （是使用MapReduce作业来实现的）

		$ hadoop distcp hdfs://namenode1/foo hdfs://namenode2/bar
		将第一个集群的/foo 目录及其内容复制到第二个集群的/bar 目录下
		多种参数选项 -overwrite -update
        $ hadoop distcp webhdfs://namenode1:50070/foo webhdfs://namenode2:50070/bar
        解决不同集群版本不兼容问题。(如果使用旧版hftp(只读)，只能在目标集群上运行作业、还可以使用HDFS HTTP代理服务）
        
		向HDFS复制数据时，考虑集群的均衡性是相当重要的。//map任务的数量 
	
- Apache Flume将大规模流数据导入HDFS
- Apache Sqoop将结构化数据（如：RMDBS)导入HDFS（Hive数据仓库）

### Hadoop存档 ###

主要用途：减少namenode内存使用，可以用作MapReduce的输入。

	$ hadoop archive -archiveName files.har /my/files /my

以上，将/my/files中的文件存档为files.har的HAR文件，存放在/my下。  
HAR文件的组成部分：两个索引文件以及部分文件的集合（_index、_masterindex、part-0、part-1、....）。

	$ hadoop fs -ls -R har:///my/files.har

HAR文件系统建立在基础文件系统之上的（如：HDFS）。

	$ hadoop fs -ls -R har://hdfs-localhost:8020/my/files.har/my/files/dir

har URI会转换成基础文件系统的URI

	$ hadoop fs -rmr /my/files.har

对于基础文件系统来说，HAR文件是一个目录，删除时要使用 `-r`

- 新建一个存档文件会创建原始文件的一个副本  
- 与tar文件类似，但HAR文件不能被压缩  
- 不能修改HAR文件  
- HAR文件作为MapReduce的输入，尽管InputFormat能够将多个文件打包成一个MapReduce分片，但它不知道文件已经存档，所以在HAR文件中处理许多小文件时，低效。