---
title: 用户指南5分钟入门Demo
date: 2018-03-08 13:20:46
tags: sqoop2
---

# sqoop2系统入门之1:用户指南5分钟入门Demo

问题导读
```
1.如何启动Sqoop客户端？
2.如何构建连接对象（Linking Ojbect）？
3.如何新建作业对象（Job Object）？
4.如何启动作业（即作业传输）？
```
—————————

该教程将会带你体验Sqoop的基本用法。你需要安装并配置Sqoop的客户端和服务端以按照该教程做下去。安装过程可以参考链接 Installation。需要注意的是该教程的具体输出信息可能会跟你的不同，因为Sqoop一直在升级，但是所有的主要信息还是一样的。
Sqoop使用唯一的代号或连续的id来区分不同的连接器、作业和配置。我们支持通过唯一的代号或连续的数据库id来查询实体。

## 1.如何启动Sqoop客户端
使用以下命令来启动交互模式的客户端：

> sqoop2-shell

配置客户端以使用Sqoop服务器：

> sqoop:000> set server --host your.host.com --port 12000 --webapp sqoop

通过简单的检查版本来确认连接已生效：
```
sqoop:000> show version --all
client version:
  Sqoop 2.0.0-SNAPSHOT source revision 418c5f637c3f09b94ea7fc3b0a4610831373a25f
  Compiled by vbasavaraj on Mon Nov  3 08:18:21 PST 2014
server version:
  Sqoop 2.0.0-SNAPSHOT source revision 418c5f637c3f09b94ea7fc3b0a4610831373a25f
  Compiled by vbasavaraj on Mon Nov  3 08:18:21 PST 2014
API versions:
  [v1]

```
你应该会看到类似的描述信息，包括sqoop客户端构建版本，服务端构建版本和rest API所支持的版本。

你也可以在sqoop命令行下用help命令检查所有支持的命令。

```
sqoop:000> help
For information about Sqoop, visit: [url=http://sqoop.apache.org/]http://sqoop.apache.org/[/url]
 
Available commands:
  exit    (\x  ) Exit the shell
  history (\H  ) Display, manage and recall edit-line history
  help    (\h  ) Display this help message
  set     (\st ) Configure various client options and settings
  show    (\sh ) Display various objects and configuration options
  create  (\cr ) Create new object in Sqoop repository
  delete  (\d  ) Delete existing object in Sqoop repository
  update  (\up ) Update objects in Sqoop repository
  clone   (\cl ) Create new object based on existing one
  start   (\sta) Start job
  stop    (\stp) Stop job
  status  (\stu) Display status of a job
  enable  (\en ) Enable object in Sqoop repository
  disable (\di ) Disable object in Sqoop repository
```
## 2.如何构建连接对象（Linking Ojbect）

检查已经在Sqoop服务端注册的连接器

```
sqoop:000> show connector
+------------------------+----------------+------------------------------------------------------+----------------------+
|          Name          |    Version     |                        Class                         | Supported Directions |
+------------------------+----------------+------------------------------------------------------+----------------------+
| hdfs-connector         | 2.0.0-SNAPSHOT | org.apache.sqoop.connector.hdfs.HdfsConnector        | FROM/TO              |
| generic-jdbc-connector | 2.0.0-SNAPSHOT | org.apache.sqoop.connector.jdbc.GenericJdbcConnector | FROM/TO              |
+------------------------+----------------+------------------------------------------------------+----------------------+

```

我们的例子包含两个连接器。 generic-jdbc-connector 是依赖Java JDBC接口的基础连接器，用于与数据源交互。它支持大部分常见的有提供jdbc驱动的数据库。请注意，jdbc驱动需要你单独安装，Sqoop并没有集成他们，因为证书不兼容。
我们例子中统一的JDBC连接器叫 generic-jdbc-connector ，我们使用这个值为这个连接器创造新的连接对象。请注意：连接对象的名字必须唯一。
```
sqoop:000> create link -connector generic-jdbc-connector
Creating link for connector with name generic-jdbc-connector
Please fill following values to create new link object
Name: First Link
 
Link configuration
JDBC Driver Class: com.mysql.jdbc.Driver
JDBC Connection String: jdbc:mysql://mysql.server/database
Username: sqoop
Password: *****
JDBC Connection Properties:
There are currently 0 values in the map:
entry#protocol=tcp 
New link was successfully created with validation status OK name First Link
```

我们新建的连接对象被赋予分配的First Link名字

在 show connector -all 命令下，我们看到有个hdfs-connector 已经注册。我们再新建一个连接对象但这次是为hdfs-connector这个连接器。
```
sqoop:000> create link -connector hdfs-connector
Creating link for connector with name hdfs-connector
Please fill following values to create new link object
Name: Second Link
 
Link configuration
HDFS URI: hdfs://nameservice1:8020/
New link was successfully created with validation status OK and name Second Link
```
## 3.如何新建作业对象（Job Object）
连接器通过实现 From 连接来读取数据且/或 To 连接来写入数据。统一的JDBC连接器两种操作都支持。每种连接器所支持的所有操作指南可以通过 show connector -all 命令来查看。要创建作业，我们需要配置作业的 From 部分和 To 部分，两部分都应该被独一无二的连接id来标识。我们的系统中已经有2个连接了，你也可以用以下命令来新建同样的连接。

```
sqoop:000> show link --all
2 link(s) to show:
link with name First Link (Enabled: true, Created by root at 11/4/14 4:27 PM, Updated by root at 11/4/14 4:27 PM)
Using Connector with name generic-jdbc-connector
  Link configuration
    JDBC Driver Class: com.mysql.jdbc.Driver
    JDBC Connection String: jdbc:mysql://mysql.ent.cloudera.com/sqoop
    Username: sqoop
    Password:
    JDBC Connection Properties:
      protocol = tcp
link with name Second Link (Enabled: true, Created by root at 11/4/14 4:38 PM, Updated by root at 11/4/14 4:38 PM)
Using Connector with name hdfs-connector
  Link configuration
    HDFS URI: hdfs://nameservice1:8020/
```
接下来，我们可以用两个连接的名字为作业同 From 和 To 创建关联

```
sqoop:000> create job -f "First Link" -t "Second Link"
 Creating job for links with from name First Link and to name Second Link
 Please fill following values to create new job object
 Name: Sqoopy
 
 FromJob configuration
 
  Schema name:(Required)sqoop
  Table name:(Required)sqoop
  Table SQL statement:(Optional)
  Table column names:(Optional)
  Partition column name:(Optional) id
  Null value allowed for the partition column:(Optional)
  Boundary query:(Optional)
 
ToJob configuration
 
  Output format:
   0 : TEXT_FILE
   1 : SEQUENCE_FILE
  Choose: 0
  Compression format:
   0 : NONE
   1 : DEFAULT
   2 : DEFLATE
   3 : GZIP
   4 : BZIP2
   5 : LZO
   6 : LZ4
   7 : SNAPPY
   8 : CUSTOM
  Choose: 0
  Custom compression format:(Optional)
  Output directory:(Required)/root/projects/sqoop
 
  Driver Config
  Extractors:(Optional) 2
  Loaders:(Optional) 2
  New job was successfully created with validation status OK  and name jobName
```
我们新的作业对象创建时被赋予Sqoopy名字。注意，如果分区列允许空值，Sqoop至少需要2个抽取器来执行数据传输。在这个场景中，仅配置一个抽取器，Sqoop会忽略这个设置并继续以2个抽取器执行下去。

## 4.如何启动作业（即作业传输）
可以使用下面命令来启动sqoop作业。
```
sqoop:000> start job -name Sqoopy
Submission details
Job Name: Sqoopy
Server URL: http://localhost:12000/sqoop/
Created by: root
Creation date: 2014-11-04 19:43:29 PST
Lastly updated by: root
External ID: job_1412137947693_0001
  [url=http://vbsqoop-1.ent.cloudera.co]http://vbsqoop-1.ent.cloudera.co[/url] ... 1412137947693_0001/
2014-11-04 19:43:29 PST: BOOTING  - Progress is not available
```
你可以使用 status job 命令交互的检查在运行作业的状态

```
sqoop:000> status job -n Sqoopy
Submission details
Job Name: Sqoopy
Server URL: http://localhost:12000/sqoop/
Created by: root
Creation date: 2014-11-04 19:43:29 PST
Lastly updated by: root
External ID: job_1412137947693_0001
  [url=http://vbsqoop-1.ent.cloudera.co]http://vbsqoop-1.ent.cloudera.co[/url] ... 1412137947693_0001/
2014-11-04 20:09:16 PST: RUNNING  - 0.00 %
```
你也可以使用以下命令启动作业并观察运行状态
```
sqoop:000> start job -n Sqoopy -s
Submission details
Job Name: Sqoopy
Server URL: http://localhost:12000/sqoop/
Created by: root
Creation date: 2014-11-04 19:43:29 PST
Lastly updated by: root
External ID: job_1412137947693_0001
  [url=http://vbsqoop-1.ent.cloudera.co]http://vbsqoop-1.ent.cloudera.co[/url] ... 1412137947693_0001/
2014-11-04 19:43:29 PST: BOOTING  - Progress is not available
2014-11-04 19:43:39 PST: RUNNING  - 0.00 %
2014-11-04 19:43:49 PST: RUNNING  - 10.00 %
```
最后，你可以使用 stop job 命令来停止作业：
> sqoop:000> stop job -n Sqoopy

[点击此处查看英文链接](http://sqoop.apache.org/docs/1.99.7/user/Sqoop5MinutesDemo.html)

















