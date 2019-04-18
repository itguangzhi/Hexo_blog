---
title: 用户指南shell命令
date: 2018-03-08 13:25:00
tags: sqoop2
---

# sqoop2系统入门之2汇总:用户指南shell命令

问题导读
```
1.sqoop2有哪两种运行模式？
2.哪个模式，有些命令不支持？
3.sqoop2辅助命令有哪些，作用是什么？
4.set命令是否连接sqoop server？
5.show命令显示哪些信息？
6.如何使用show 命令显示指定信息？
7.sqoop2中，如何定义数据源及数据流向？
8.你认为link的作用是什么？
```
sqoop2对于sqoop1有很大的变化，但是网上并没有系统的文章，所以这里about云整理下。以下内容来自官网，及个人理解，如有错误或则异议，大家可回帖讨论。
Sqoop 2提供的命令是通过使用REST 接口进行交互。客户端能运行两种模式：交互和批处理模式。 create, update 和clone命令在批处理模式中当前不支持。交互模式支持所有的命令。

可以使用sqoop2-shell，进入交互模式

> sqoop2-shell

批处理模式需要额外的参数，需要添加上script.sqoop的路径

> sqoop2-shell /path/to/your/script.sqoop

Sqoop client 脚本包括Sqoop client 命令，以#开头，表示注释。如下面例子
```
# Specify company server
set server --host sqoop2.company.net

# Executing given job
start job --name 1
```
## 1. 资源文件
Sqoop 2 客户端可以加载资源文件。Sqoop 刚开始执行的时候，会检测当前用户的home目录是否有.sqoop2rc文件.如果存在，它将被执行。这个文件会被加载到交互模式和批处理模式。它将用于执行批处理模式的兼容命令。

资源文件例子：
```
# Configure our Sqoop 2 server automatically
set server --host sqoop2.company.net

# Run in verbose mode by default
set option --name verbose --value true
```
## 2. 命令
Sqoop 2包含几种命令。每一个命令可能有一个以上函数接受不同的参数。并不是所有的命令都支持交互模式和批处理模式。

### 2.1 辅助命令
辅助命令是改善用户体验并纯粹在客户端运行的命令。因此，他们不需要连接到服务器。

**exit** ：退出客户端。也可以使用EOT 字符。
**history** ：可以看到以前执行的命令
**help** ：显示所有可用的命令

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
### 2.2 set命令

Set命令允许设置客户端属性，类似辅助命令，不需要链接Sqoop server。Set命令不用于重新配置Sqoop server

可用函数：
```
函数		描述
-------------------------
server	设置server连接配置
选项		设置客户端选项
```
#### 2.2.1 Set Server 功能
配置连接Sqoop server-host端口和web应用程序名。参数：

```
参数				默认值			描述
-h, --host		localhost		Server name (FQDN)，Sqoop server 运行在什么地方
-p, --port		12000			TCP 端口
-w, --webapp	sqoop			Jetty应用程序名
-u, --url						Sqoop Server的url的格式
```
例子：

> set server --host sqoop2.company.net --port 80 --webapp sqoop

或则

> set server --url [url=http://sqoop2.company.net:80/sqoop]http://sqoop2.company.net:80/sqoop[/url]

**注意**：
如果给定了--url，那么--host, --port 或则--webapp将会被忽略

#### 2.2.2Set 选项功能

配置Sqoop 客户端相关选项，这个函数有两个必须的参数name 和value.名称表示内部属性名称，值包含应设置的新值。选项名如下：
```
选项名			默认值			描述
verbose			false		启用verbose模式m客户端会输出额外的信息
poll-timeout	10000		Server poll 超时时间,单位为毫秒
```
例子：

> set option --name verbose --value true

> set option --name poll-timeout --value 20000

### 2.3 show命令
show命令显示如下各种信息。可用的功能：
```
功能			描述
server		显示连接信息到sqoopserver
option		显示客户端选项
version		显示构建的版本，-all选项显示server构建版本和支持的api版本
connector	显示连接配置和它的相关配置
driver		显示驱动配置和它的相关配置
link		显示sqoop link
job			显示sqoop jobs
```

#### 2.3.1Show Server 功能
显示 Sqoop server连接的详细信息
```
参数						描述
-a, --all        	显示连接相关信息（host, 端口, 应用程序名）
-h, --host        	显示主机
-p, --port        	显示端口
-w, --webapp        显示应用程序名
```
例子：

> show server --all

#### 2.3.2 Show Option 功能
显示客户端选项值，这个功能当没有参数的时，会显示所有客户端信息。

```
参数					描述
-n, --name        	显示客户端给定的值
```
可检测 Set 选项功能部分，得到支持的option名称

例子：

> show option --name verbose

#### 2.3.3 Show 版本功能
显示构建的客户端和服务端的版本，以及支持的 rest api 版本.

```
参数					描述
-a, --all        	显示所有信息（服务端，客户端，api）
-c, --client        显示客户端版本
-s, --server        显示服务端版本号
-p, --api			显示支持api版本
```
例子:

> show version --all

#### 2.3.4Show Connector 功能
显示持久化链接配置和相关配置，用于创建相关的 link 和job objects

```
参数					描述Connector 的信息
-a, --all        	显示所有
-c, --cid <x>       显示带有id的链接的信息
```
例子：

> show connector --all or show connector

#### 2.3.5Show Driver 功能
显示持久化驱动配置和他的相关配置用于创建 job objects.这个功能没有其它参数。只有一个注册的驱动在sqoop。
例子:

> show driver

#### 2.3.6Show Link 功能
显示link持久化对象
```
参数					描述
-a, --all        	显示所有有效link
-n, --name <x>      显示指定名字的link
```
例子：

> show link --all or show link --name linkName


#### 2.3.7 Show job功能
显示持久化job 对象
```
参数					描述
-a, --all        	显示所有有效job
-n, --name <x>      显示带有名字的job
```

#### 2.3.8 Show Submission 功能
显示持久化job submission对象
```
参数						描述
-j, --job <x>        	显示提交job的有效submission
-d, --detail        	显示提交hob的所有详细信息
```
例子：
```
show submission
show submission --j jobName
show submission --job jobName --detail
```
### 2.4 create命令

创建新的link和job对象。这个命令仅支持交互模式。在分别创建link和job对象时，会要求用户输入link配置和job的from /to和驱动配置

可用功能
```
功能			描述
link	创建新的link对象
job		创建新的hob对象
```


#### 2.4.1 create link功能

创建新的link对象
```
参数							描述
-c, --connector <x>        	创建名字为<x>的link对象
```

例子

> create link --connector connectorName or create link -c connectorName

#### 2.4.2 create Job 功能

创建新的job对象

```
参数						描述
-f, --from <x>        	创建新的名字为<x>的from link的job对象
-t, --to <t>        	创建新的名字为<t>的to link的job对象
```

例子：

> create job --from fromLinkName --to toLinkName or create job --f fromLinkName --t toLinkName

这里也是sqoop2与sqoop1区别最大的地方：
sqoop1是自己指定的，而sqoop2则是先定义link然后，链接两个link.

### 2.5 update命令

更新命令仅在交互模式下支持，允许编辑link和job对象。

#### 2.5.1更新link功能


更新link对象
```
参数						描述
-n, --name <x>        	更新名字为<x>的link

```
例子：

> update link --name linkName

#### 2.5.2更新job功能

```
参数						描述
-n, --name <x>        	更新名字为<x>的job
```

例子：

> update job --name jobName

### 2.6 delete命令


从sqoop server删除link和job

#### 2.6.1 删除link功能

删除link
```
参数						描述
-n, --name <x>        	删除名字为link的对象
```

例子：

> delete link --name linkName


#### 2.6.1删除job功能


删除job
```
参数						描述
-n, --name <x>        	删除名字为<x>的job
```

例子：

> delete job --name jobName



### 2.7 clone命令

Clone 名字从sqoop server加载已存在的link和job.允许用户就地更新，这样会创建新的link或则job。这个命令不支持批处理模式。

#### 2.7.1 clone link功能
Clone已经存在的link
```
参数	描述
-n, --name <x>        	Clone名字为<x>link
```
例子

> clone link --name linkName


#### 2.7.2 clone job功能
clone已经存在的job.
```
参数						描述
-n, --name <x>        	clone名字为<x>的job
```
例子：

> clone job --name jobName

### 2.8start命令

start命令执行现有的Sqoop job.

#### 2.8.1 start job功能

start job，启动 已经运行的job被视为无效。
```
参数						描述
-n, --name <x>        	启动名字为 <x>的job
-s, --synchronous       同步作业执行
```

例子：

> start job --name jobName

> start job --name jobName --synchronous



### 2.9 stop命令

中断作业执行

```
参数						描述
-n, --name <x>        	中断名字为<x>的job
```

例子：

> stop job --name jobName



### 3.0 status命令


状态命令将检索作业的最后状态。

#### 3.0.1 job状态功能

检索给定job的状态

```
参数						描述
-n, --name <x>        	检索名字为<x>的job的状态
```

> status job --name jobName

> Next  Previous

