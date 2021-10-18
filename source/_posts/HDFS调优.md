---
title: HDFS调优
date: 2021-10-18 10:46:29
tags:
---
<meta name="referrer" content="no-referrer" />

# NameNode和DataNnode内存分配
在Hadoop3.X版本中，NameNode和DataNode的内存默认是自动分配的

`hadoop-env.sh`文件中有
```shell

# 如果没有default，则JVM会根据机器内存情况自动分配
# 并且，也说明了进程会优先使用各自在_OPT的值

# The maximum amount of heap to use (Java -Xmx).  If no unit
# is provided, it will be converted to MB.  Daemons will
# prefer any Xmx setting in their respective _OPT variable.
# There is no default; the JVM will autoscale based upon machine
# memory size.
# export HADOOP_HEAPSIZE_MAX=

# The minimum amount of heap to use (Java -Xms).  If no unit
# is provided, it will be converted to MB.  Daemons will
# prefer any Xms setting in their respective _OPT variable.
# There is no default; the JVM will autoscale based upon machine
# memory size.
# export HADOOP_HEAPSIZE_MIN=

```

通常，NameNode和DataNode的内存值不采用自动分配，而是根据实际情况手动分配
```shell
# 设置JVM选项，这些设置将会被添加到HADOOP_OPTS变量中，而且会覆盖，选择default那一个进行添加


# Specify the JVM options to be used when starting the NameNode.
# These options will be appended to the options specified as HADOOP_OPTS
# and therefore may override any similar flags set in HADOOP_OPTS
#
# a) Set JMX options
# export HDFS_NAMENODE_OPTS="-Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=1026"
#
# b) Set garbage collection logs
# export HDFS_NAMENODE_OPTS="${HADOOP_GC_SETTINGS} -Xloggc:${HADOOP_LOG_DIR}/gc-rm.log-$(date +'%Y%m%d%H%M')"
#
# c) ... or set them directly
# export HDFS_NAMENODE_OPTS="-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:${HADOOP_LOG_DIR}/gc-rm.log-$(date +'%Y%m%d%H%M')"

# this is the default:
# export HDFS_NAMENODE_OPTS="-Dhadoop.security.logger=INFO,RFAS"

#设置NAMENODE的内存为1G
export HDFS_NAMENODE_OPTS="-Dhadoop.security.logger=INFO,RFAS -Xmx1024m"


#DATANODE的设置一样

# DataNode specific parameters
# Specify the JVM options to be used when starting the DataNode.
# These options will be appended to the options specified as HADOOP_OPTS
# and therefore may override any similar flags set in HADOOP_OPTS
#
# This is the default:
# export HDFS_DATANODE_OPTS="-Dhadoop.security.logger=ERROR,RFAS"

# 设置DATANODE的内存为1G
export HDFS_DATANODE_OPTS="-Dhadoop.security.logger=ERROR,RFAS -Xmx1024"

```
可以通过命令`jmap -heap 进程号`查看内存使用情况，其中进程号可通过jps查看NameNode或者DataNode的进程。

# NameNode心跳并发配置
NameNode有一个工作线程池，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作
`hdfs-site.xml`中配置如下，默认是10，经验设置:20*LOG(#cluster size)，其中#cluster size为机器台数。
```xml
<property>
  <name>dfs.namenode.handler.count</name>
  <value>10</value>
  <description>The number of server threads for the namenode.</description>
</property>
```

# 回收站配置
开启回收站功能，可以将删除的文件在不超时的情况下，恢复原数据，起到防止误删除，备份等作用

* fs.trash.interval表示设置文件的存活时间，默认为0，即禁止。
* fs.trash.checkpoint.interval：检查回收站的间隔时间，如果该值为0，则该值设置和fs.trash.interval的值一致。

`core-site.xml`配置
```xml
<property>
    <name>fs.trash.interval</name>
    <value>0</value>
    <description>Number of minutes after which the checkpoint gets deleted. If zero, the trash feature is disabled. This option may be configured both on the server and the client. If trash is disabled server side then the client side configuration is checked. If trash is enabled on the server side then the value configured on the server is used and the client configuration value is ignored.
    </description>
</property>


<property>
    <name>fs.trash.checkpoint.interval</name>
    <value>0</value>
    <description>Number of minutes between trash checkpoints. Should be smaller or equal to fs.trash.interval. If zero, the value is set to the value of fs.trash.interval. Every time the checkpointer runs it creates a new checkpoint out of current and removes checkpoints created more than fs.trash.interval minutes ago.
    </description>
</property>
```

回收站在HDFS集群中的路径：~/.Trash

通过网页删除的文件不会进入到回收站

通过程序删除的文件不会进入回收站，需要调用`moveToTrash()`才进入回收站

只有在命令行利用`hadoop fs -rm`命令删除的文件才会进入到回收站，如果想恢复的话，就把回收站的文件`hadoop fs -mv`到其他地方即可。

# HDFS压测

HDFS的读写性能主要受网络和磁盘影响比较大

## 测试HDFS写性能

假设有三台服务器，每台服务器4核，则测试为：
```shell
hadoop jar /opt/module/hadoop-3.3.1/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.3.1-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB

```
表示10个文件，每个文件128MB，写操作。

通常测试文件个数=集群CPU总核数-1。而且-nrFiles的数值也是mapTask的数量。

程序跑完之后，会有以下结果：
```
Number of files:10     文件个数
Total MBytes processed:1280   文件总大小
Throughput mb/sec:1.61    总吞吐量：（所有文件数量）/（总时间）
Average IO ratemb/sec: 1.9    （map1平均速度+...+map10平均速度）/ 10
IO rate std deviation: 0.76    IO的方差
Test exec time sec: 133.05     总的执行时间
```

测试结果分析：

如果副本1就在集群的机器1上，则一共参与测试的文件就为：10个文件 * 2个副本 = 20个文件，2个副本的原因为：副本1本来就在机器1上，只是机器2和机器3需要副本。

如果Throughput mb/sec:1.61，则实测速度为：1.61 * 20个文件 = 32 M/s，如果三台服务器的带宽都为12.5M/s，则总带宽为：12.5+12.5+12.5 = 37.5M/s，则三台服务器的网络资源大体上都已经用满

如果文件在客户端，并不在这三台服务器上，则一共参与测试的文件就为：10个文件*3个副本 = 30个文件。后续计算跟上面一致

如果实测速度远小于网络，并且实测速度不能满足工作需求，则可以考虑采用固态硬盘或者增加硬盘个数

## 测试HDFS读性能
```shell
hadoop jar /opt/module/hadoop-3.3.1/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.3.1-tests.jar TestDFSIO -read -nrFiles 3 -fileSize 128MB
```

读数据并不会每个机器都读一遍，而是采用“就近原则”，选择最近的机器将文件读下来。如果客户端就在集群机器1上，则会出现不占用网络资源，读取速度比网络资源快的情况







