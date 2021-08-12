---
title: hadoop基本框架学习
date: 2021-06-08 17:13:11
tags: [hadoop]
categories: hadoop
---

<meta name="referrer" content="no-referrer" />

# Hadoop基本框架

# HDFS
HDFS是Hadoop Distributed File System(分布式文件系统)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gruj58bijpj60pw0h4q3e02.jpg)


HDFS采用master/slave架构。

一个HDFS集群是由一个Namenode和一定数目的Datanodes组成。

Namenode是一个中心服务器，负责管理文件系统的名字空间(namespace)以及客户端对文件的访问

集群中的Datanode一般是一个节点一个，负责管理它所在节点上的存储

HDFS暴露了文件系统的名字空间，用户能够以文件的形式在上面存储数据。从内部看，一个文件其实被分成一个或多个数据块，这些块存储在一组Datanode上。Namenode执行文件系统的名字空间操作，比如打开、关闭、重命名文件或目录。它也负责确定数据块到具体Datanode节点的映射。Datanode负责处理文件系统客户端的读写请求。在Namenode的统一调度下进行数据块的创建、删除和复制。

一个典型的**部署场景**是一台机器上只运行一个Namenode实例，而集群中的其它机器分别运行一个Datanode实例。这种架构并不排斥在一台机器上运行多个Datanode，只不过这样的情况比较少见。

集群中单一Namenode的结构大大简化了系统的架构。Namenode是所有HDFS元数据的仲裁者和管理者，这样，用户数据永远不会流过Namenode。

HDFS支持传统的层次型文件组织结构。用户或者应用程序可以创建目录，然后将文件保存在这些目录里。文件系统名字空间的层次结构和大多数现有的文件系统类似：用户可以创建、删除、移动或重命名文件。

Namenode负责维护文件系统的名字空间，任何对文件系统名字空间或属性的修改都将被Namenode记录下来。应用程序可以设置HDFS保存的文件的副本数目。文件副本的数目称为文件的副本系数，这个信息也是由Namenode保存的。

HDFS被设计成能够在一个大集群中跨机器可靠地存储超大文件。它将每个文件存储成一系列的数据块，除了最后一个，所有的数据块都是同样大小的。为了容错，文件的所有数据块都会有副本。每个文件的数据块大小和副本系数都是可配置的。应用程序可以指定某个文件的副本数目。副本系数可以在文件创建的时候指定，也可以在之后改变。HDFS中的文件都是一次性写入的，并且严格要求在任何时候只能有一个写入者。

Namenode全权管理数据块的复制，它周期性地从集群中的每个Datanode接收心跳信号和块状态报告(Blockreport)。接收到心跳信号意味着该Datanode节点工作正常。块状态报告包含了一个该Datanode上所有数据块的列表。



# YARN

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1grum00cmedj30l30gctim.jpg)


## ResourceManager: 全局资源管理和任务调度

负责全局的资源管理和任务调度，把整个集群当成计算资源池，只关注分配，不管应用，且不负责容错

### 资源管理
以前资源是每个节点分成一个个的Map slot和Reduce slot，现在是一个个Container，每个Container可以根据需要运行ApplicationMaster、Map、Reduce或者任意的程序

以前的资源分配是静态的，目前是动态的，资源利用率更高

Container是资源申请的单位，一个资源申请格式：`<resource-name, priority, resource-requirement, number-of-containers>, resource-name`：主机名、机架名或*（代表任意机器）, resource-requirement：目前只支持CPU和内存

用户提交作业到ResourceManager，然后在某个NodeManager上分配一个Container来运行ApplicationMaster，ApplicationMaster再根据自身程序需要向ResourceManager申请资源

YARN有一套Container的生命周期管理机制，而ApplicationMaster和其Container之间的管理是应用程序自己定义的





![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1grunnt7wi6j30dv0fvq7s.jpg)


### 任务调度

只关注资源的使用情况，根据需求合理分配资源

Scheluer可以根据申请的需要，在特定的机器上申请特定的资源（ApplicationMaster负责申请资源时的数据本地化的考虑，ResourceManager将尽量满足其申请需求，在指定的机器上分配Container，从而减少数据移动）


### 内部结构

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gruo2838tdj30gk0c7dlq.jpg)


* Client Service: 应用提交、终止、输出信息（应用、队列、集群等的状态信息）

* Adaminstration Service: 队列、节点、Client权限管理

* ApplicationMasterService: 注册、终止ApplicationMaster, 获取ApplicationMaster的资源申请或取消的请求，并将其异步地传给Scheduler, 单线程处理

* ApplicationMaster Liveliness Monitor: 接收ApplicationMaster的心跳消息，如果某个ApplicationMaster在一定时间内没有发送心跳，则被任务失效，其资源将会被回收，然后ResourceManager会重新分配一个ApplicationMaster运行该应用（默认尝试2次）

* Resource Tracker Service: 注册节点, 接收各注册节点的心跳消息

* NodeManagers Liveliness Monitor: 监控每个节点的心跳消息，如果长时间没有收到心跳消息，则认为该节点无效, 同时所有在该节点上的Container都标记成无效，也不会调度任务到该节点运行

* ApplicationManager: 管理应用程序，记录和管理已完成的应用

* ApplicationMaster Launcher: 一个应用提交后，负责与NodeManager交互，分配Container并加载ApplicationMaster，也负责终止或销毁

* YarnScheduler: 资源调度分配， 有FIFO(with Priority)，Fair，Capacity方式

* ContainerAllocationExpirer: 管理已分配但没有启用的Container，超过一定时间则将其回收





## NodeManager: 单个节点的资源管理和监控

Node节点下的Container管理

启动时向ResourceManager注册并定时发送心跳消息，等待ResourceManager的指令

监控Container的运行，维护Container的生命周期，监控Container的资源使用情况

启动或停止Container，管理任务运行时的依赖包（根据ApplicationMaster的需要，启动Container之前将需要的程序及其依赖包、配置文件等拷贝到本地）

### 内部结构
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gruofr52ipj30kw0f0qbs.jpg)

* NodeStatusUpdater: 启动向ResourceManager注册，报告该节点的可用资源情况，通信的端口和后续状态的维护

* ContainerManager: 接收RPC请求（启动、停止），资源本地化（下载应用需要的资源到本地，根据需要共享这些资源）

PUBLIC: /filecache

PRIVATE: /usercache//filecache

APPLICATION: /usercache//appcache//（在程序完成后会被删除）

* ContainersLauncher: 加载或终止Container
* ContainerMonitor: 监控Container的运行和资源使用情况
* ContainerExecutor: 和底层操作系统交互，加载要运行的程序



## ApplicationMaster: 单个作业的资源管理和任务监控
具体功能描述：

* 计算应用的资源需求，资源可以是静态或动态计算的，静态的一般是Client申请时就指定了，动态则需要ApplicationMaster根据应用的运行状态来决定
* 根据数据来申请对应位置的资源（Data Locality）
* 向ResourceManager申请资源，与NodeManager交互进行程序的运行和监控，监控申请的资源的使用情况，监控作业进度
* 跟踪任务状态和进度，定时向ResourceManager发送心跳消息，报告资源的使用情况和应用的进度信息
* 负责本作业内的任务的容错

ApplicationMaster可以是用任何语言编写的程序，它和ResourceManager和NodeManager之间是通过ProtocolBuf交互，以前是一个全局的JobTracker负责的，现在每个作业都一个，可伸缩性更强，至少不会因为作业太多，造成JobTracker瓶颈。同时将作业的逻辑放到一个独立的ApplicationMaster中，使得灵活性更加高，每个作业都可以有自己的处理方式，不用绑定到MapReduce的处理模式上

### 如何计算资源需求
一般的MapReduce是根据block数量来定Map和Reduce的计算数量，然后一般的Map或Reduce就占用一个Container

### 如何发现数据的本地化
数据本地化是通过HDFS的block分片信息获取的

## Container: 资源申请的单位和任务运行的容器
* 基本的资源单位（CPU、内存等）
* Container可以加载任意程序，而且不限于Java
* 一个Node可以包含多个Container，也可以是一个大的Container
* ApplicationMaster可以根据需要，动态申请和释放Container


## YARN基本流程


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1grunu6spixj30f70e4tbz.jpg)



1. Job submission

从ResourceManager 中获取一个Application ID 检查作业输出配置，计算输入分片 拷贝作业资源（job jar、配置文件、分片信息）到 HDFS，以便后面任务的执行

2. Job initialization

ResourceManager 将作业递交给 Scheduler（有很多调度算法，一般是根据优先级）Scheduler 为作业分配一个 Container，ResourceManager 就加载一个 application master process 并交给 NodeManager。

管理 ApplicationMaster 主要是创建一系列的监控进程来跟踪作业的进度，同时获取输入分片，为每一个分片创建一个 Map task 和相应的 reduce task Application Master 还决定如何运行作业，如果作业很小（可配置），则直接在同一个 JVM 下运行

3. Task assignment

ApplicationMaster 向 Resource Manager 申请资源（一个个的Container，指定任务分配的资源要求）一般是根据 data locality 来分配资源

4. Task execution

ApplicationMaster 根据 ResourceManager 的分配情况，在对应的 NodeManager 中启动 Container 从 HDFS 中读取任务所需资源（job jar，配置文件等），然后执行该任务



![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1grunuxvflij30er0d9mzi.jpg)

5. Progress and status update

定时将任务的进度和状态报告给 ApplicationMaster Client 定时向 ApplicationMaster 获取整个任务的进度和状态

6. Job completion

Client定时检查整个作业是否完成 作业完成后，会清空临时文件、目录等



# MapReduce
