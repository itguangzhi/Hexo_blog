---
title: 6-开发指南SqoopJava客户端API指南
date: 2018-03-14 17:16:59
tags:sqoop2
---
此文档解释如何使用Sqoop Java客户端API与外部应用交互。使用客户端API可以达成跟sqoop命令一样的效果。需要Sqoop客户端JAR包及对应的依赖。
以下是包含了将所有支持的操作封装成的方法的主类

```
public class SqoopClient {
  ...
}
```

文中的Java 客户端API 演示选用统一JDBC连接器。在运行使用sqoop客户端API的应用程序时，确保sqoop服务器处于运行状态。

# 1. 工作流
以下是在Sqoop服务器执行sqoop作业的工作流。

* 1.根据给定的连接器名字创建连接对象- 创建连接对象并返回

* 2.根据给定的“from” 和“to”连接名字创建作业- 创建作业并返回

* 3.根据给定的作业名字启动作业-在服务端启动作业并新增一个作业提交记录

# 2.项目依赖
以下是maven 依赖

```
<dependency>
  <groupId>org.apache.sqoop</groupId>
    <artifactId>sqoop-client</artifactId>
    <version>${requestedVersion}</version>
</dependency>
```

3. 初始化
使用给的的server url入参初始化SqoopClient类

```
String url = "http://localhost:12000/sqoop/";
SqoopClient client = new SqoopClient(url);
Server URL value can be modfied by setting value to setServerUrl(String) method
client.setServerUrl(newUrl);
```

# 4. 连接
连接器拥有与多个数据源交互的功能，所以在Sqoop中可以被用来传输多个数据源数据。注册的连接器实现将会提供逻辑来读取或（且）写入所代表的数据源。一个连接器可以有一个或多个连接相关联。使用Java客户端API可以创建、更新或删除关联到已注册的连接器的连接。创建或更新连接需要你为专门的连接器生成连接配置。因此，首先要做的事是获取已注册的连接器列表并选择一个连接器来创建连接与之关联。然后你可以用《7.显示连接的配置和输入名称》(见本文第7节)获取该连接器所有的配置和输入。

## 4.1. 保存连接
先调用方法createLink(connectorName) ，为方法传入连接器名称来创建一个新的连接，该方法会返回MLink类型的对象，该对象包含连接器的id和空的连接配置输入。然后在配置输入中填入相关的值。通过调用saveLink 来传递填充过的MLink对象。

```
/ 为连接创建一个占位符
MLink link = client.createLink("connectorName");
link.setName("Vampire");
link.setCreationUser("Buffy");
MLinkConfig linkConfig = link.getConnectorLinkConfig();
//填入连接配置的值
linkConfig.getStringInput("linkConfig.connectionString").setValue("jdbc:mysql://localhost/my");
linkConfig.getStringInput("linkConfig.jdbcDriver").setValue("com.mysql.jdbc.Driver");
linkConfig.getStringInput("linkConfig.username").setValue("root");
linkConfig.getStringInput("linkConfig.password").setValue("root");
// 保存填充过的连接对象
Status status = client.saveLink(link);
if(status.canProceed()) {
System.out.println("Created Link with Link Name : " + link.getName());
} else {
System.out.println("Something went wrong creating the link");
}
```

status.canProceed() 返回true如果状态是OK或WARNING. 在发送状态前，通过与连接配置输入对应的验证器来验证连接对象的配置值。
一旦执行成功，新的连接名字被分配给连接对象，否则就抛出异常。 link.getName() 方法返回已经在sqoop库持久化的该对象的独一无二的名称。
用户可以使用下列方法取到一个连接。
Method        	Description
getLink(linkName)   	通过名字返回一个连接
getLinks()   	返回sqoop的连接列表
     
# 5. 作业
一个sqoop作业负责数据传输（从From 数据源到To数据源）的From 和To 部分。 From 和 To 都被他们对应的连接器连接id唯一标识。举个例子，在新建一个作业的时候，我们需要指定FromLinkId 和ToLinkId 。因此，创建作业的前提条件是首先要新建以上提及的两个连接。
一旦指定了 From 和To 的连接名字，那么对应的连接对象的连接器的作业配置需要被填充值。你可以用《7.显示连接的配置和输入名称》(见本文第7节)获取该连接器所有的配置和输入。一个连接器可以有一个或多个连接。然后我们用 From 到To 方向的连接来生成对应的 MFromConfig  和MToConfig ，以此类推。
除了填充代表连接的From 和 To 的作业配置，还需要填充控制作业执行引擎环境的驱动配置。例如，如果作业的执行引擎碰巧是MapReduce，那么我们就要配置从From数据源读取数据的mapper数量。 

## 5.1. 保存作业
以下代码创建并保存一个作业

```
String url = "http://localhost:12000/sqoop/";
SqoopClient client = new SqoopClient(url);
//新建一个job对象Creating dummy job object
MJob job = client.createJob("fromLinkName", "toLinkName");
job.setName("Vampire");
job.setCreationUser("Buffy");
//设置"FROM"连接作业配置的值
MFromConfig fromJobConfig = job.getFromJobConfig();
fromJobConfig.getStringInput("fromJobConfig.schemaName").setValue("sqoop");
fromJobConfig.getStringInput("fromJobConfig.tableName").setValue("sqoop");
fromJobConfig.getStringInput("fromJobConfig.partitionColumn").setValue("id");
//设置"TO"连接作业配置的值
MToConfig toJobConfig = job.getToJobConfig();
toJobConfig.getStringInput("toJobConfig.outputDirectory").setValue("/usr/tmp");
//设置驱动配置的值
MDriverConfig driverConfig = job.getDriverConfig();
driverConfig.getStringInput("throttlingConfig.numExtractors").setValue("3");
 
Status status = client.saveJob(job);
if(status.canProceed()) {
 System.out.println("Created Job with Job Name: "+ job.getName());
} else {
 System.out.println("Something went wrong creating the job");
}
```

用户可以使用以下方法来取到一个作业。
Method        	Description
getJob(jobName)	通过名字返回作业
getJobs()        	返回sqoop的作业列表
        

## 5.2.状态码列表
Function        	Description
OK	没有异常或警告。
WARNING	验证过的实体足够正确，能保证继续运行。没有严重错误。
ERROR	验证过的实体有严重错误。除非报告的异常已经解决，否则无法继续运行。
        

## 5.3. 查看验证消息是否有错误或警告
一旦有WARNING 和ERROR状态，用户需要逐条查看验证消息。

```
printMessage(link.getConnectorLinkConfig().getConfigs());
 
private static void printMessage(List<MConfig> configs) {
  for(MConfig config : configs) {
    List<MInput<?>> inputlist = config.getInputs();
    if (config.getValidationMessages() != null) {
     // print every validation message
     for(Message message : config.getValidationMessages()) {
      System.out.println("Config validation message: " + message.getMessage());
     }
    }
    for (MInput minput : inputlist) {
      if (minput.getValidationStatus() == Status.WARNING) {
       for(Message message : minput.getValidationMessages()) {
        System.out.println("Config Input Validation Warning: " + message.getMessage());
      }
    }
    else if (minput.getValidationStatus() == Status.ERROR) {
      for(Message message : minput.getValidationMessages()) {
       System.out.println("Config Input Validation Error: " + message.getMessage());
      }
     }
    }
   }
```

## 5.4. 更新连接和作业
在库中建好连接或作业后，可以使用以下方法更新或删除连接或作业

Method        	Description
updateLink(link)	更新连接并检查状态码是否包含错误或警告。
deleteLink(linkName)        	删除连接。只有在要删除的连接没有被任务作业使用的前提才能删除。
updateJob(job)        	更新作业并检查状态码是否包含错误或警告。
deleteJob(jobName)        	删除作业

# 6.启动作业
启动作业需要作业名字。一旦启动成功，getStatus()方法将会返回“BOOTING”或“RUNNING”信息

```
//启动作业
MSubmission submission = client.startJob("jobName");
System.out.println("Job Submission Status : " + submission.getStatus());
if(submission.getStatus().isRunning() && submission.getProgress() != -1) {
  System.out.println("Progress : " + String.format("%.2f %%", submission.getProgress() * 100));
}
System.out.println("Hadoop job id :" + submission.getExternalId());
System.out.println("Job link : " + submission.getExternalLink());
Counters counters = submission.getCounters();
if(counters != null) {
  System.out.println("Counters:");
  for(CounterGroup group : counters) {
    System.out.print("\t");
    System.out.println(group.getName());
    for(Counter counter : group) {
      System.out.print("\t\t");
      System.out.print(counter.getName());
      System.out.print(": ");
      System.out.println(counter.getValue());
    }
  }
}
if(submission.getExceptionInfo() != null) {
  System.out.println("Exception info : " +submission.getExceptionInfo());
}
 
 
//检查正在运行作业的状态
MSubmission submission = client.getJobStatus("jobName");
if(submission.getStatus().isRunning() && submission.getProgress() != -1) {
  System.out.println("Progress : " + String.format("%.2f %%", submission.getProgress() * 100));
}
 
//停止正在运行的作业
submission.stopJob("jobName");
```
以上的代码块中，作业的启动是异步的。要同步启动作业，使用startJob(jobName, callback, pollTime) 方法。如果你对获取作业的状态不感兴趣，可以调用相同方法，但callback参数值设为null，这样它就会返回最终的作业状态。 pollTime 是从sqoop服务器获取作业状态的请求间隔时间，其值必须大于0。如果pollTime设的值太小，我们就会频繁的请求sqoop服务器。当同步作业启动且callback参数值不为null，如果启动成功，它先触发callback的 submitted(MSubmission) 方法，在每个poll time时间间隔结束后，它又会触发的 updated(MSubmission) 方法并最终触发callback API 的 finished(MSubmission) 方法来结束作业。

7.显示连接的配置和输入名称
可以展示每个连接器的连接和作业配置类型的配置/输入名称

```
String url = "http://localhost:12000/sqoop/";
SqoopClient client = new SqoopClient(url);
String connectorName = "connectorName";
//连接器的连接配置
describe(client.getConnector(connectorName).getLinkConfig().getConfigs(), client.getConnectorConfigBundle(connectorName));
// 连接器的from作业配置
describe(client.getConnector(connectorName).getFromConfig().getConfigs(), client.getConnectorConfigBundle(connectorName));
// 连接器的to作业配置
describe(client.getConnector(connectorName).getToConfig().getConfigs(), client.getConnectorConfigBundle(connectorName));
 
void describe(List<MConfig> configs, ResourceBundle resource) {
  for (MConfig config : configs) {
    System.out.println(resource.getString(config.getLabelKey())+":");
    List<MInput<?>> inputs = config.getInputs();
    for (MInput input : inputs) {
      System.out.println(resource.getString(input.getLabelKey()) + " : " + input.getValue());
    }
    System.out.println();
  }
}
```
以上的Sqoop 2客户端API教程展示了如何新建连接、新建作业及启动作业