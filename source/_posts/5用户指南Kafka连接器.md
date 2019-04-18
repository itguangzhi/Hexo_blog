---
title: '5:用户指南Kafka连接器'
date: 2018-03-14 17:11:59
tags: sqoop2
---
目前，只支持TO作业。

# 1. 用法
通过创建连接器连接(link)和使用该连接的作业(job)来使用该Kafka连接器。

## 1.1. 配置连接
配置连接(link)涉及的输入包括：
Input        	Type        	Description        	Example

Broker list	String        	必须的。逗号隔开的kafka 服务器。	example.com:10000,example.com:11000
Zookeeper connection        	String        	必须的。逗号隔开的zookeeper服务器quorum。	/etc/conf/hadoop

## 1.2. TO 作业配置
与作业配置相关联的FROM作业的输入包括：
  Input                 	Type                  	Description                  	Example
topic	String	必须的。要转移到的kafka主题。	my topic

## 2. 加载器（loader）
在数据加载阶段，每个加载器都直接写数据到Kafka。写入kafka的数据是不保证顺序性的。