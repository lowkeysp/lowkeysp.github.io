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

# DataNode配置多目录
DataNode可以配置成多个目录，每个目录存储的数据不一样（数据不是副本）

`hdfs-site.xml`配置
```xml
<property>
  <name>dfs.datanode.data.dir</name>
  <value>file://${hadoop.tmp.dir}/dfs/data1,file://${hadoop.tmp.dir}/dfs/data2</value>
  <description>Determines where on the local filesystem an DFS data node should store its blocks. If this is a comma-delimited list of directories, then data will be stored in all named directories, typically on different devices. Directories that do not exist are ignored.</description>
</property>
```
# 磁盘间数据均衡
生产环境，由于硬盘空间不足，往往需要增加一块硬盘，刚加载的硬盘没有数据时，可以执行磁盘数据均衡命令（hadoop3.x特性）

生成均衡计划
```
hdfs diskbalancer -plan 机器地址
```

执行均衡计划
```
hdfs diskbalancer -execute 机器地址.plan.json
```

查看当前均衡任务执行情况
```
hdfs diskbalancer -query 机器地址
```

取消均衡任务
```
hdfs diskbalancer -cancel 机器地址.plan.json
```

# HDFS集群扩容以及缩容
## 白名单和黑名单创建
可以在白名单中配置ip地址，在白名单中的ip地址所代表的node可以用来存储数据，否则不能进行存储数据

黑名单中配置ip地址，表示不能用来存储数据。

配置如下：

* 在NameNode节点的`/opt/module/hadoop-3.3.1/etc/hadoop`目录下创建whitelist和blacklist
* 在`hdfs-site.xml`配置文件中，指定白黑名单地址
```xml
<property>
  <name>dfs.hosts</name>
  <value>/opt/module/hadoop-3.3.1/etc/hadoop/whitelist</value>
  <description>白名单</description>
</property>

<property>
  <name>dfs.hosts.exclude</name>
  <value>/opt/module/hadoop-3.3.1/etc/hadoop/blacklist</value>
  <description>黑名单</description>
</property>

```
* 第一次添加白名单和黑名单需要重启集群，如果不是第一次，则只需要刷新NameNode即可，`hdfs dfsadmin -refreshNodes`
 

# 节点间的数据均衡
* 开启数据均衡
```shell
sbin/start-balancer.sh -threshold 10

# 其中，-threshold 10表示集群中各个节点的磁盘空间利用率相差不超过10%
```
* 关闭数据均衡
```shell
sbin/stop-balancer.sh
```
由于HDFS需要启动单独的Rebalance Server来执行Rebalance操作，所以尽量不要在NameNode上执行start-balancer.sh命令，而是找一台比较空闲的机器

# 纠删码原理
## 纠删码介绍
HDFS(Hadoop Distributed File System)为了保证数据的可靠性默认数据存储策略是3副本，即在写入数据的时候，会占用该数据大小3倍的空间。这样就造成了大量的空间浪费。对此，HDFS引入在RAID磁盘阵列中已应用成熟的技术：EC（Erasure Coding，纠删码）。

EC的原理就是，通过将1个文件打散成很小的数据块（如128k、256k、1M等），然后将这些数据块打散存储于多个DN中，然后在另外的若干个DN中存储EC编解码算法生成的校验块。如果某个存储文件块的DN不可用后，将可以通过其他DN中存储的数据块和校验块反向计算出来这个丢失的数据块。

## EC策略说明
不同的EC编解码算法、数据块大小、数据块和校验块个数，可以构成不同的EC策略。下面以RS-6-3-1024k这种策略说明下EC策略的定义：
* 使用RS（Reed Solomon）编解码算法
* 有6个原始数据块分别存储在6个DN节点，3个校验块分别存储在3个DN节点上
* 最大可以容忍3个块丢失的异常情况
* 每个文件块的大小为1024k（即1MB）
* 如果使用该EC策略存储的文件为100MB，则写入DataNode中的总数据量为(1+3/6) * 100MB=150MB。其中:数据块总大小为文件大小100MB;校验块总大小为3/6 * 100MB=50MB

使用EC存储后，可以在提供相同可靠性的前提下，节省大量的存储空间。例如，要达到3副本相同的可靠性，仅需要1.5倍的原文件大小即可，从而节约一半的存储空间（原先需要3倍的原文件大小）

HDFS引入了EC特性后，在减少磁盘使用量时，当然也引入了一些问题与限制：
* 文件在存储时被打散到多个DN中，文件读取不再有“本地读”的概念，导致了数据失去亲和性，引发了非常严重的数据倾斜
* 写入文件时，都需要计算（生成校验块、或通过校验块校验），这会消耗更多CPU，使复杂度增高
* 在DN节点不可用时，需要通过计算得到缺失的数据块，这会消耗更多CPU，使复杂度增高、异常恢复时间更长
* 客户端在读写文件时，需要同时连接所有涉及的DN读写数据块和校验块
* 因为实现的原因，目前不支持2个基本的FileSystem接口：append()、truncate()，调用会直接抛异常。不同EC策略的文件的concat()，也会抛错

针对这些限制和问题，我们可以得出，该特性适用于一次写入大量数据的场景，不适用于以下的场景
* 大量小文件
* 打开后长期持续写入的文件
* 需要调用append()或truncate()接口修改的文件
* 相比副本存储，EC存储对系统有如下影响：
* NameNode、DataNode、客户端都需要消耗更多的CPU、内存、网络资源。
* DataNode异常后，需要更长时间来恢复该DataNode上的数据，同时也会降低相关文件的读取性能。
* 针对这些影响，在准备使用EC的集群上，建议根据业务负载进行如下操作：
* 调大NameNode、DataNode的“GC_OPTS”中内存参数（主要是-Xmx的值）。
* 调大HDFS客户端及依赖于HDFS的上层组件的内存参数。
* 调大DataNode的数据连接线程数参数“dfs.datanode.max.transfer.threads”


## EC相关命令
列出当前集群中所有支持的EC编解码算法,`hdfs ec -listCodecs`
```
[root@10-219-254-200 ~]#hdfs ec -listCodecs
Erasure Coding Codecs: Codec [Coder List]
RS [RS_NATIVE, RS_JAVA]
RS-LEGACY [RS-LEGACY_JAVA]
XOR [XOR_NATIVE, XOR_JAVA]
```

列出当前集群中所有的EC策略,`hdfs ec -listPolicies`
```
[root@10-219-254-200 ~]#hdfs ec -listPolicies
Erasure Coding Policies:
ErasureCodingPolicy=[Name=RS-10-4-1024k, Schema=[ECSchema=[Codec=rs, numDataUnits=10, numParityUnits=4]],
CellSize=1048576, Id=5], State=DISABLED
ErasureCodingPolicy=[Name=RS-3-2-1024k, Schema=[ECSchema=[Codec=rs, numDataUnits=3, numParityUnits=2]],
CellSize=1048576, Id=2], State=DISABLED
ErasureCodingPolicy=[Name=RS-6-3-1024k, Schema=[ECSchema=[Codec=rs, numDataUnits=6, numParityUnits=3]],
CellSize=1048576, Id=1], State=ENABLED
ErasureCodingPolicy=[Name=RS-LEGACY-6-3-1024k, Schema=[ECSchema=[Codec=rs-legacy, numDataUnits=6, numParityUnits=3]],
CellSize=1048576, Id=3], State=DISABLED
ErasureCodingPolicy=[Name=XOR-2-1-1024k, Schema=[ECSchema=[Codec=xor, numDataUnits=2, numParityUnits=1]],
CellSize=1048576, Id=4], State=ENABLED
```

启用指定的EC策略,`hdfs ec -enablePolicy -policy <policyName>`
```
[root@10-219-254-200 ~]#hdfs ec -enablePolicy -policy RS-3-2-1024kErasure coding policy RS-3-2-1024k is enabled
```

**只有启用（ENABLED）状态的EC策略，才能设置给目录**

停用指定的EC策略,`hdfs ec -disablePolicy -policy <policyName>`
```
[root@10-219-254-200 ~]#hdfs ec -disablePolicy -policy RS-3-2-1024kErasure coding policy RS-3-2-1024k is disabled
```

新增自定义EC策略`hdfs ec -addPolicies -policyFile <配置文件路径>`,需要先编写自定义EC策略的配置文件（可以参考 <HDFS客户端安装目录>/HDFS/hadoop/etc/hadoop/user_ec_policies.xml.template 样例来编写）,自定义添加的EC策略，默认是停用（DISABLED）状态的，需要启用后才能使用,样例内容如下：
```xml
<?xml version="1.0"?> 
 <configuration> 
 <!-- The version of EC policy XML file format, it must be an integer --> 
 <layoutversion>1</layoutversion> 
 <schemas> 
   <!-- schema id is only used to reference internally in this document --> 
   <schema id="XORk2m1"> 
     <!-- The combination of codec, k, m and options as the schema ID, defines 
      a unique schema, for example 'xor-2-1'. schema ID is case insensitive --> 
     <!-- codec with this specific name should exist already in this system --> 
     <codec>xor</codec> 
     <k>2</k> 
     <m>1</m> 
     <options> </options> 
   </schema> 
   <schema id="RSk12m4"> 
     <codec>rs</codec> 
     <k>12</k> 
     <m>4</m> 
     <options> </options> 
   </schema> 
   <schema id="RS-legacyk12m4"> 
     <codec>rs-legacy</codec> 
     <k>12</k> 
     <m>4</m> 
     <options> </options> 
   </schema> 
 </schemas> 
 <policies> 
   <policy> 
     <!-- the combination of schema ID and cellsize(in unit k) defines a unique 
      policy, for example 'xor-2-1-256k', case insensitive --> 
     <!-- schema is referred by its id --> 
     <schema>XORk2m1</schema> 
     <!-- cellsize must be an positive integer multiple of 1024(1k) --> 
     <!-- maximum cellsize is defined by 'dfs.namenode.ec.policies.max.cellsize' property --> 
     <cellsize>131072</cellsize> 
   </policy> 
   <policy> 
     <schema>RS-legacyk12m4</schema> 
     <cellsize>262144</cellsize> 
   </policy> 
 </policies> 
 </configuration>
```

删除指定的EC策略`hdfs ec -removePolicy -policy <policy>`,
```
[root@10-219-254-200 ~]#hdfs ec -removePolicy -policy XOR-2-1-128kErasure coding policy XOR-2-1-128k is removed

```

设置目录的EC策略`hdfs ec -setPolicy -path <path> [-policy <policy>] [-replicate]`,
```
[root@10-219-254-200 ~]#hdfs ec -setPolicy -path /test -policy RS-6-3-1024k
Set RS-6-3-1024k erasure coding policy on /test
Warning: setting erasure coding policy on a non-empty directory will not automatically convert existing files to
RS-6-3-1024k erasure coding policy
```
* 给一个目录设置EC策略后，该目录下已有文件将不受影响（存储方式不变），而新创建的文件将按照设置的EC策略来存储;
* -replicate参数，用来给目录设置3副本存储策略，而非EC策略;
* -replicate 和 -policy <policyName>不能同时使用

获取指定目录的EC策略,`hdfs ec -getPolicy -path <path>`
```
[root@10-219-254-200 ~]#hdfs ec -getPolicy -path /test RS-6-3-1024k

```

取消指定目录的EC策略`hdfs ec -unsetPolicy -path <path>`
```
root@10-219-254-200 ~]#hdfs ec -unsetPolicy -path /test
Unset erasure coding policy from /test
Warning: unsetting erasure coding policy on a non-empty directory will not automatically convert existing files to
replicated data.
```
* 当一个目录被取消EC策略后，如果其父目录有EC策略，则将继承其父目录的EC策略

## EC与三副本转换
如果Distcp的源文件有EC文件时，需要针对不同的需求，添加不同的参数：
* 如果在拷贝过程中，保持文件的EC策略，需要在Distcp时加入参数：“-preserveec”。
* 如果不需要保持源文件的EC策略，而按照目的目录的EC策略（或副本策略）来将文件重写到目的目录，则需要在Distcp时加入参数：“-skipcrccheck”。

如果不加这两个参数之一，则不同EC策略的文件，会因为其校验值不同而导致Distcp任务失败。

### 副本文件转换成EC文件
此处以把“/src”目录下的所有文件全部文件转换为“RS-6-3-1024k”策略的EC文件为例。
* 1、创建一个临时目录（如“/tmp/convert_tmp_dir”，必须是不存在的目录），用于临时存储转换完成的数据。
    
    `hdfs dfs -mkdir /tmp/convert_tmp_dir`

* 2、启用EC策略“ RS-6-3-1024k”（如果已启用，可以跳过此步骤）。

    `hdfs ec -enablePolicy -policy RS-6-3-1024k`

* 3、设置“/tmp/convert_tmp_dir”的EC策略为“RS-6-3-1024k”。

    `hdfs ec -setPolicy -path /tmp/convert_tmp_dir -policy RS-6-3-1024k`

* 4、使用Distcp拷贝文件，此时所有文件会按照“RS-6-3-1024k”策略写入临时目录。

    `hadoop distcp -numListstatusThreads 40 -update -delete -skipcrccheck  /src /tmp/convert_tmp_dir`

* 5、移动“/src”为“/src-to-delete”。

    `hdfs dfs -mv /src /src-to-delete`

* 6、移动“/tmp/convert_tmp_dir”为“/src”。

    `hdfs dfs -mv /tmp/convert_tmp_dir /src`

* 7、删除“/src-to-delete”。

    `hdfs dfs -rm -r -f /src-to-delete`

### EC文件转换成副本文件
此处以把“/src”目录下的所有文件全部文件转换为副本文件为例。

* 1、创建一个临时目录（如“/tmp/convert_tmp_dir”，必须是不存在的目录），用于临时存储转换完成的数据。

    `hdfs dfs -mkdir /tmp/convert_tmp_dir`

* 2、设置“/tmp/convert_tmp_dir”为副本策略。

    `hdfs ec -setPolicy -path /tmp/convert_tmp_dir -replicate`

* 3、使用Distcp拷贝文件，此时所有文件会按照副本策略写入临时目录。

    `hadoop distcp -numListstatusThreads 40 -update -delete -skipcrccheck  /src /tmp/convert_tmp_dir`

* 4、移动“/src”为“/src-to-delete”。

    `hdfs dfs -mv /src /src-to-delete`

* 5、移动“/tmp/convert_tmp_dir”为“/src”。

    `hdfs dfs -mv /tmp/convert_tmp_dir /src`

* 6、删除“/src-to-delete”。

    `hdfs dfs -rm -r -f /src-to-delete`

# 异构存储
## 异构存储类型
* RAM_DISK：内存，当之无愧是最快的，但官方有句提醒：We have observed that the latency overhead from network replication negates the benefits of writing to memory. 也就是说因为网络的延迟抵消写入内存带给我们的速度。经过测试的实际情况是，这种配置方式和SSD可能差不了太多。
* SSD：固态磁盘，主要用来存储热数据，即访问频繁的数据。
* DISK：普通的磁盘，例如，SATA盘。
* ARCHIVE：是一种支持PB级的高容量存储，主要用于归档数据使用，有很小的计算能力，而我们所谓的冷数据适合使用archive存储类型。

## 异构存储策略
|策略ID|策略名称|副本分布|解释|
|---|---|--- |---|
|15|Lazy_Persist|RAM_DISK:1,DISK:n-1|一个副本保存在RAM_DISK中，其余保存在磁盘中|
|12|ALL_SSD|SSD:n|所有副本保存在SSD中|
|10|One_SSD|SSD:1,DISK:n-1|一个副本保存在SSD中，其余保存在磁盘中|
|7|Hot|DISK:n|所有副本保存在磁盘中，默认策略|
|5|Warm|DISK:1,ARCHIVE:n-1|一个副本保存在磁盘中，其余保存在归档存储中|
|2|Cold|ARCHIVE:n|所有副本保存在归档存储中|


## shell命令
* 查看当前有哪些存储策略可以用
```shell
hdfs storagepolicies –listPolicies
Block Storage Policies:
    BlockStoragePolicy{COLD:2, storageTypes=[ARCHIVE], creationFallbacks=[], replicationFallbacks=[]}
    BlockStoragePolicy{WARM:5, storageTypes=[DISK, ARCHIVE], creationFallbacks=[DISK, ARCHIVE], replicationFallbacks=[DISK, ARCHIVE]}
    BlockStoragePolicy{HOT:7, storageTypes=[DISK], creationFallbacks=[], replicationFallbacks=[ARCHIVE]}
    BlockStoragePolicy{ONE_SSD:10, storageTypes=[SSD, DISK], creationFallbacks=[SSD, DISK], replicationFallbacks=[SSD, DISK]}
    BlockStoragePolicy{ALL_SSD:12, storageTypes=[SSD], creationFallbacks=[DISK], replicationFallbacks=[DISK]}
    BlockStoragePolicy{LAZY_PERSIST:15, storageTypes=[RAM_DISK, DISK], creationFallbacks=[DISK], replicationFallbacks=[DISK]}
```
* 指定路径的储存策略 文件目录均可
```
hdfs storagepolicies -setStoragePolicy -path /xxxx -policy ONE_SSD
```
* 获取指定路径的存储策略
```
hdfs storagepolicies -getStoragePolicy -path /xxxx
```
* 取消存储策略
```
hdfs storagepolicies -unsetStoragePolicy -path /xxxx
```
* MOVER定期扫描HDFS文件，检查文件的存放是否符合它自身的存储策略。如果数据块不符合自己的策略，它会把数据移动到该去的地方
```
hdfs mover [-p <files/dirs> | -f <local file name>]
-p  指定要迁移的文件/目录，多个以空格分隔
​
-f  指定本地一个文件路径，该文件列出了需要迁移的文件或者目录（一个一行）
​
如果不指定参数，那么就移动根目录。
```

## 配置异构存储
在hdfs-site.xml 的配置属性dfs.datanode.data.dir 中进行本地对应存储目录的设置，同时带上一个存储类型标签,声明此目录用的是哪种类型的存储介质，存储标签有`[SSD] [DISK] [ARCHIVE] [RAM_DISK]`这4种中的任何一种，如果目录前没有带上,则默认是DISK类型
```xml
# 此参数用于启用或禁用异构存储策略，其默认值为"true",即允许用户更改文件和目录的存储策略
<property>
    <name>dfs.storage.policy.enabled</name>
    <value>true</value>
</property>
# 此参数表示用，分隔的目录列表，用来存储hdfs的数据，对于HDFS存储策略，应使用相应的存储类型
#（[SSD] / [DISK] / [ARCHIVE] / [RAM_DISK]）标记目录。如果不加存储类型，则默认为[DISK]类型
<property>
    <name>dfs.datanode.data.dir</name>
    <value>[SSD]file:///hdfsdata/ssd0,[SSD]file:///hdfsdata/ssd1,file:///hdfsdata/sata0,
file:///hdfsdata/sata1</value>
</property>
```

# 安全模式
安全模式是NameNode的维护状态，在安全模式下，文件系统只能read-only，不能删除，复制，修改数据块。

当NameNode启动时，会自动进入安全模式，在安全模式下，会执行以下任务：
* NameNode reads the FsImage and EditLog from disk, applies all the transactions from the EditLog to the in-memory representation of the FsImage, and flushes out this new version into a new FsImage on disk. （NameNode在加载镜像文件和编辑日志期间处于安全模式）
* Receive block reports from the DataNodes in the cluster（接收DataNodes注册期间处于安全模式）

在安全模式下，当你执行写操作时，会抛出异常“SafeModeException”，并说明：Name node is in safe mode。

离开安全模式的条件：
* NameNode首先需要确认（现有的副本数/总的副本数 > threshold-pct),比如系统有1000个副本，NameNode启动后，通过各个DataNode上报，发现丢了10个副本，只有990个副本了，则这个数就为0.99,threshold-pct可以在`hdfs-site.xml`文件中配置`dfs.namenode.safemode.threshold-pct`，默认0.999f。
* threshold满足后，还需要有一个稳定时间，过了这个时间，才能真正离开安全模式，该时间仍可在`hdfs-site.xml`文件中`dfs.namenode.safemode.extension`配置，默认是30000，即30秒。

## 安全模式命令行
* 进入安全模式
```
hdfs dfsadmin -safemode enter
```
* 退出安全模式
```
hdfs dfsadmin -safemode leave
```
* 获得安全模式状态
```
hdfs dfsadmin -safemode get
```
* If you want any file operation command to block till HDFS exists safemode
```
 hdfs dfsadmin -safemode wait
```
* 强制退出安全模式
```
hdfs dfsadmin -safemode forceExit
```

当如果磁盘损坏导致threshold-pct达不到要求，从而一直处于安全模式后，该如何处理？

如果数据比较重要的话，需要找专人进行磁盘修复

如果数据不重要且不能恢复的话，则可以先用命令行退出安全模式，然后再把系统检测出来的丢失的数据删除即可。

# 磁盘读写性能

可用`fio`命令检测磁盘的读写性能是否处于正常状态

参考：https://linux.cn/article-9912-1.html

# 小文件
小文件一般指的是明显小于HDFS块大小（默认是128M）的文件。

HDFS中的文件、目录、块信息等都会以对象的形式存储在NameNode内存中，每一个对象大概占用150byte，因此，如果小文件过多，则会占用大量的NameNode内存。举个例子，假如有10000000个文件，每个文件用一个block，则NameNode需要使用3G内存存储这些对象信息。这样namenode内存容量严重制约了集群的扩展。其次，访问大量小文件速度远远小于访问几个大文件。HDFS最初是为流式访问大文件开发的，如果访问大量小文件，需要不断的从一个datanode跳到另一个datanode，严重影响性能。最后，处理大量小文件速度远远小于处理同等大小的大文件的速度。每一个小文件要占用一个slot，而task启动将耗费大量时间甚至大部分时间都耗费在启动task和释放task上。

Hadoop自带的解决小文件的方法有：Hadoop Archives (HAR files) ，Sequence Files，HBase。

Hadoop Archive或者HAR，是一个高效地将小文件放入HDFS块中的文件存档工具，它能够将多个小文件打包成一个HAR文件，这样在减少namenode内存使用的同时，仍然允许对文件进行透明的访问。

HAR的操作：
* 比如将`/foo/bar`下的所有小文件存档成到`/outputdir/`，并命名为`zoo.har`
```
hadoop archive -archiveName zoo.har -p /foo/bar /outputdir
```
* 查看HAR文件存档中的文件
```
hadoop fs -ls har://outputdir/zoo.har
```
* 解压HAR文件
```
hadoop fs -cp har://outputdir/zoo.har/* /
```

使用HAR时需要两点，第一，对小文件进行存档后，原文件并不会自动被删除，需要用户自己删除；第二，创建HAR文件的过程实际上是在运行一个mapreduce作业，因而需要有一个hadoop集群运行此命令

此外，HAR还有一些缺陷：第一，一旦创建，Archives便不可改变。要增加或移除里面的文件，必须重新创建归档文件。第二，要归档的文件名中不能有空格，否则会抛出异常，可以将空格用其他符号替换(使用-Dhar.space.replacement.enable=true 和-Dhar.space.replacement参数)。

# distcp命令实现两个Apache Hadoop之间的数据迁移
```
bash$ hadoop distcp hdfs://nn1:8020/foo/bar \
                    hdfs://nn2:8020/bar/foo
```
这条命令会把nn1集群的/foo/bar目录下的所有文件或目录名展开并存储到一个临时文件中，这些文件内容的拷贝工作被分配给多个map任务， 然后每个TaskTracker分别执行从nn1到nn2的拷贝操作。注意DistCp使用绝对路径进行操作。

命令行中可以指定多个源目录：
```
bash$ hadoop distcp hdfs://nn1:8020/foo/a \
                    hdfs://nn1:8020/foo/b \
                    hdfs://nn2:8020/bar/foo
```
或者使用-f选项，从文件里获得多个源：
```
bash$ hadoop distcp -f hdfs://nn1:8020/srclist \
                       hdfs://nn2:8020/bar/foo
```
其中srclist 的内容是
```
    hdfs://nn1:8020/foo/a
    hdfs://nn1:8020/foo/b
```

# Uber模式

当MapReduce任务提交后，ResourceManager会创建一个container，开启ApplicationMaster进程（对于MapReduce来说，ApplicationMaster就是MRAppMaster）。ApplicationMaster会获取输入切片的个数，并基于此，以及配置，决定开启多少个mapTask和reduceTask.

如果不开启Uber模式，则mapTask和reduceTask也会分别创建container，运行各自的进行。而如果开启了Uber模式，所有的mapTasks和reduceTasks将会在ApplicationMaster所在的container中运行，也就是说整个MR作业运行的过程只会启动ApplicationMaster container，因为不需要启动mapper 和 reducer containers.

启用uber模式的要求非常严格，代码如下:
```
isUber = uberEnabled && smallNumMapTasks && smallNumReduceTasks
    && smallInput && smallMemory && smallCpu 
    && notChainJob && isValidUberMaxReduces;
```
* uberEnabled：其实就是 mapreduce.job.ubertask.enable 参数的值，默认情况下为 false ；也就是说默认情况不启用Uber模式
* smallNumMapTasks：启用Uber模式的作业Map的个数必须小于等于 mapreduce.job.ubertask.maxmaps 参数的值，该值默认为9；也计算说，在默认情况下，如果你想启用Uber模式，作业的Map个数必须小于10
* smallNumReduceTasks：同理，Uber模式的作业Reduce的个数必须小于等于mapreduce.job.ubertask.maxreduces，该值默认为1；也计算说，在默认情况下，如果你想启用Uber模式，作业的Reduce个数必须小于等于1
* smallInput：不是任何作业都适合启用Uber模式的，输入数据的大小必须小于等于 mapreduce.job.ubertask.maxbytes 参数的值，默认情况是HDFS一个文件块大小
* smallMemory：因为作业是在AM所在的container中运行，所以要求我们设置的Map内存（mapreduce.map.memory.mb）和Reduce内存（mapreduce.reduce.memory.mb）必须小于等于 AM所在容器内存大小设置（yarn.app.mapreduce.am.resource.mb）
* smallCpu：同理，Map配置的vcores（mapreduce.map.cpu.vcores）个数和 Reduce配置的vcores（mapreduce.reduce.cpu.vcores）个数也必须小于等于AM所在容器vcores个数的设置（yarn.app.mapreduce.am.resource.cpu-vcores）
* notChainJob：此外，处理数据的Map class（mapreduce.job.map.class）和Reduce class（mapreduce.job.reduce.class）必须不是 ChainMapper 或 ChainReducer 才行；
* isValidUberMaxReduces：目前仅当Reduce的个数小于等于1的作业才能启用Uber模式。

这些参数在`mapred-site.xml`中可设置






