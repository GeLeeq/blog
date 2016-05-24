---
layout: post
title:  Spark Quick Start
date:	2016-5-24 17:14:40  
category: "Spark"
---

2009诞生、2010开源、2011高级组件、2013交给Apache基金会、2014年5月正式发布Spark1.0  

Spark1.4.0 起添加了R语言支持、支持Python3

### Spark是什么 ###

最原始的Spark: 数据处理引擎 （相对于MapReduce)  

通过在一个统一的框架下支持不同的分布式计算（包括：批处理、迭代算法、交互式数据操作、流处理...）

![Saprk软件栈](http://geleeq.github.io/blog/post_res/images/spark/spark_soft_stack.jpg)

Spark Core  
  
- 任务调度、内存管理、错误恢复、与存储系统交互等模块, spark core还包含了对弹性分布式数据集`RDD`的API定义。

		RDD
			在Spark中，我们通过对分布式数据集的操作来表达我们的计算意图，这些计算会自动地在集群上
		并行进行。这样的数据集被称为弹性分布式数据集(resilient distributed dataset)

spark各高级组件密切结合设计的优点：

- 软件栈中所有的程序库和高级组件都可以从下层的改进中获益
- 运行整个软件栈的代价降低（部署、维护、测试、支持...）
- 我们能够构建出无缝整合不同处理模型的应用（如：将数据流中的数据使用机器学习算法进行实时分类）

### Spark shell ###

	bin/pyspark #Python
	>>> lines = sc.textFile("README.md") # 创建一个名为lines的RDD
	>>> lines.count() # 统计RDD中的元素个数
	>>> lines.first()

	bin/spark-shell #Scala

### Spark应用程序 ###
![](http://geleeq.github.io/blog/post_res/images/spark/spark-exec.jpg)

通过一个驱动器程序创建一个SparkContext对象来访问Saprk，这个对象代表对计算集群的一个连接，及创建一系列RDD，
然后进行并行操作。

>前面例子里，实现的驱动器程序就是Spark shell本身，shell启动时已经自动创建了一个SparkContext对象，叫作sc的变量

独立程序中使用Spark

java ---> Maven

scala ---> sbt或Gradle

步骤：  
1、初始化SparkContext对象  
2、操作RDD  
3、stop()或system.exit(0)

>提交：

>bin/spark-submit --class cn.nubia.xxx.WordCount ./target/xxx.jar

>spark-submit脚本可以为我们配置Spark需要的环境变量