---
title: '4:用户指南HDFS连接器'
date: 2018-03-14 17:04:37
tags:sqoop2
---

# 1. 用法
通过创建连接器连接(link)和使用该连接的作业(job)来使用该HDFS连接器。


## 1.1. 配置连接(link)
配置连接(link)涉及的输入包括：
Input        	Type        	Description        	Example

URI	String	可选的。HDFS 文件系统的URI。参考以下注意点。	hdfs://example.com:8020/

Configuration directory	String	可选的。集群配置目录的路径。	/etc/conf/hadoop

###1.1.1. 注意点
另外指定的URI将会覆盖配置中声明的URI。


## 1.2. FROM 作业配置
FROM作业涉及的输入包括：
Input        	Type        	Description        	Example

Input directory	String	必须的。连接器将会根据该HDFS路径查找文件。参考以下注意点。	/tmp/sqoop2/hdfs

Null value	String	可选的。从文件中抽取到的空值用该值代替。参考以下注意点。	N

Override null value	Boolean	可选的。连接器根据该值判断是否替换空值。参考以下注意点。	true

### 1.2.1. 注意点
Input directory 内的所有文件都会被抽取。

Null value 和override null value是配合使用的。如果override null value没有设置为true，那么在抽取数据时候就不会使用null value。


## 1.3. TO 作业配置
TO作业涉及的输入包括：

Input                 	Type                 	Description                 	
Example
  Output directory	String	必须的。连接器将会根据该HDFS路径查找文件。参考以下注意点。	/tmp/sqoop2/hdfs

Output format	Enum	可选。将数据输出时采用的数据格式。参考以下注意点。	CSV

Compression	Enum	可选。压缩类。参考以下注意点。	GZIP
  Custom compression	String	  可选。自定义的压缩类。完整类路径。	org.apache.sqoop.SqoopCompression

Null value	String	可选的。从文件中抽取到的空值用该值代替。参考以下注意点。	N
  Override null value	Boolean	可选的。连接器根据该值判断是否替换空值。参考以下注意点。	true

Append mode	Boolean	可选的。追加到已有的文件目录。	true

### 1.3.1. 注意点

* Output format 当前只支持CSV。

* Compression 支持所有Hadoop 压缩类。.
* Null value 和override null value是配合使用的。如果override null value没有设置为true，那么在抽取数据时候就不会使用null value。

# 2. 分区器Partitioner 
HDFS 连接器的分区器根据指定的输入目录中所有文件的文件块总数将数据分区。文件块将会根据所在的节点和机架划分到不同的分片。

# 3. 抽取器(Extractor)
在抽取阶段，文件系统API被用于查询HDFS中的文件。使用的HDFS 集群通过以下来定义:
连接配置中的HDFS URI。
连接配置中的Hadoop 配置。
抽取框架所使用的Hadoop配置。


文件格式必须为CSV。CSV文件中的空值可以通过null value设置。例如：

```
1,\N
2,null
3,NULL
```

在上面例子中，如果null value 设置为N ，那么第一行的NULL值就会被转译。

# 4.加载器（Loader）
在数据加载阶段，使用文件系统API写入HDFS 。新建的文件数等于运行的加载器数目。CSV文件中的空值可以通过null value设置。例如：

Id        	Value
1	NULL
2	value
如果null value 设置为N ， 那在HDFS中的数据就会像:

```
1,\N
2,value
```

# 5. 销毁器（Destroyers）
HDFS TO作业的销毁器将所有新建的文件移动到适当的输出路径。 

