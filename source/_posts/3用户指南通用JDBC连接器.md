---
title: '3:用户指南通用JDBC连接器'
date: 2018-03-14 16:42:12
tags:sqoop2
---
通用JDBC连接器可以连接所有支持JDBC4规范的数据源。
# 1.用法

通过创建连接器连接(link)和使用该连接的作业(job)来使用该通用JDBC连接器。

## 1.1配置连接(link)

配置连接(link)涉及的输入包括：

Input        Type        Description        Example
JDBC Driver Class
String
必须的。完整的jdbc类路径。配置且Sqoop服务器需要能找到该类。
com.mysql.jdbc.Driver

JDBC Connection String
String
用于连接数据源的Jdbc连接字符串。创建时的连接的连通性是可选的。
jdbc:mysql://localhost/test

Username
String
可选的。连接数据源时提供的用户名。创建时的连通性的可选的。
sqoop

Password
String
可选的。连接数据源时提供的密码。创建时的连通性的可选的。
sqoop

JDBC Connection Properties
Map
可选的。传递到jdbc驱动的 Jdbc连接属性集合。
profileSQL=true&useFastDateParsing=false

## 1.2FROM 作业配置
FROM作业涉及的输入包括：

Input        	Type        Description        	Example
Schema name	String	可选。作为表的一部分，表中不可或缺的模式名	sqoop

Table name	String	可选。从该表名导入数据。参考以下注意点。	test

Table SQL statement	String	可选。用于执行任意形式查询的sql statement。	SELECT COUNT(*) FROM test ${CONDITIONS}

Table column names	String	可选。从jdbc数据源要抽取的列。列之间用逗号隔开。	col1,col2

Partition column name	Map	可选。根据该列名将数据传输进程进行分区。默认选择主键的第一列。	col1

Null value allowed for the partition column	Boolean	可选。根据分区列是否允许有null值来选择true 或 false 。	true

Boundary query	String	可选。用于分区时定义上下界的查询。	


### 1.2.1. 注意点
> 1.表名和表的sql statement是互斥的。配置了表名，就不用配置表的sql statement。配置了表的sql statement，就不用配置表名。
> 2.只有配置了表名，才可以配置表的列名。
3.If there are columns with similar names, column aliases are required.如果有相同的列名，给列名赋别名是必须的。

例如：SELECT table1.id as "i", table2.id as "j" FROM table1 INNER JOIN table2 ON table1.id = table2.id.

## 1.3.  TO 作业配置
TO作业涉及的输入包括：
Input        	Type        	Description        	Example

Schema name	String	可选。表中不可或缺的模式名。	sqoop

Table name	String	可选。从该表名导入数据。参考以下注意点。	test

Table SQL statement	String	可选。用于执行任意形式查询的sql statement。	INSERT INTO test (col1, col2) VALUES (?, ?)

Table column names	String	可选。要插入的数据源的列。列之间用逗号隔开。	col1,col2

Stage table name	String	可选，用于作为stage表的表名。	staging

Should clear stage table	Boolean	可选。根据stage表的列是否允许有null值来选择true 或 false 。	true

### 1.3.1. 注意点：
表名和表的sql statement是互斥的。配置了表名，就不用配置表的sql statement。配置了表的sql statement，就不用配置表名。
只有配置了表名，才可以配置表的列名。

## 2.2. 分区器Partitioner
统一jdbc连接器的分区器用于生成条件供抽取器使用。根据分区列不同的数据类型，它分区数据传输的方式也不同。但是，每种策略都大致使用以下形式：

```
(上界 – 下界) / (最大分区数)
```
在没指定别的列的情况下，默认使用主键分区数据。
目前支持以下的数据类型:

```
       TINYINT
       SMALLINT
       INTEGER
       BIGINT
       REAL
       FLOAT
       DOUBLE
       NUMERIC
       DECIMAL
       BIT
       BOOLEAN
       DATE
       TIME
       TIMESTAMP
       CHAR
       VARCHAR
       LONGVARCHAR
```
## 2.3. 抽取器(Extractor)
在抽取阶段，使用SQL查询JDBC数据源。配置不同，SQL也不同。

* 如果指定了表名，将会使用

```
SELECT * FROM <table name>
```
形式生成SQL statement

* 如果指定了表名及列名，将会使用

```
SELECT <columns> FROM <table name>
```

形式
生成SQL statement。

* 如果指定了表的SQL statement ，就会使用该SQL statement。

分区器生成的条件将会被追加到SQL查询语句后以查询部分数据。

统一JDBC连接器抽取CSV数据并以CSV媒介数据类型提供使用。

## 2.4. 加载器（Loader）
在数据加载阶段，使用SQL查询JDBC数据源。配置不同，SQL也不同。

* 如果指定了表名，将会使用

```
INSERT INTO <table name> (col1, col2, ...) VALUES (?,?,..)
```

形式生成SQL statement
* 如果指定了表名及列名，将会使用

```
INSERT INTO <table name> (<columns>) VALUES (?,?,..)
```
生成SQL statement。
如果指定了表的SQL statement ，就会使用该SQL statement。
统一JDBC连接器抽取CSV数据并以CSV媒介数据类型提供使用。

## 2.5. 销毁器（Destroyers）
统一JDBC连接器在TO作业的销毁器中执行两步操作：
* 将stage表的内容拷贝到目标表。
* 清空stagin表.
对FROM作业不执行操作