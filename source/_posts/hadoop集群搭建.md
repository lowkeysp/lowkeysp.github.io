---
title: hadoop集群搭建
date: 2021-07-02 22:10:24
tags:
---

<meta name="referrer" content="no-referrer" />

可以使用VMWARE PRO软件，创建若干个Linux虚拟机，进行Hadoop的集群搭建


使用VMWARE创建虚拟机后，需要通过xshell远程登陆linux服务器，具体操作如下：

# xshell远程登陆linux服务器

## 设置静态IP地址

在`/etc/netplan/`目录下，有一个yaml文件
```
lowkeysp-00@lowkeysp-00:/etc/netplan$ ls
01-network-manager-all.yaml

```

对该文件进行编辑，修改
```shell
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:            #通过ifconfig查看网卡
      dhcp4: no        # 不适用DHCP，使用静态
      addresses: [192.168.10.1/24]    # IP地址/掩码
      gateway4: 192.168.10.2      # 网关地址
      nameservers: 
          addresses: [192.168.10.2]    # DNS地址

```

编辑完成后，
```shell
sudo netplan apply
```
之后，可以通过`ifconfig`查看确实已经更改为静态地址
```shell
lowkeysp-00@lowkeysp-00:/etc/netplan$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.1  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fef8:63de  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:f8:63:de  txqueuelen 1000  (以太网)
        RX packets 552  bytes 53961 (53.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 515  bytes 66516 (66.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (本地环回)
        RX packets 182  bytes 14978 (14.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 182  bytes 14978 (14.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```


# 配置hostname和hosts文件

```
# 配置hostname，该例子使用了三台虚拟机，分别为lowkeysp-01,lowkeysp-03,lowkeysp-04，则在相应的`/etc/hostname`下配置
lowkeysp-01

# 配置`/etc/hosts`,。添加
lowkeysp-00@lowkeysp-01:/opt/module/hadoop-3.3.1$ cat /etc/hosts

192.168.10.1  lowkeysp-01
192.168.10.3  lowkeysp-03
192.168.10.4  lowkeysp-04

```


## 修改VMWare的地址
【编辑】-【虚拟网络编辑器】

选中 VMnet8 那一行，点击右下角的【更改设置】

之后，将下方的子网IP改为：192.168.10.0，再点击【NAT设置】，网关IP改为：192.168.10.2，点击确定，再点击应用

## 修改Win系统地址

【网络和Internet设置】-【更改适配器选项】

选择【VMWare Network Adapter VMNet8】，右键-【属性】，【Internet协议版本4】，点击属性，修改IP地址为：192.168.10.101，默认网关是192.168.10.2，DNS服务器为：8.8.8.8，点击确定


## 在ubuntu上装SSH服务器，用于SSH登陆
```
sudo apt-get install openssh-server
```

## 使用xshell，用ssh方式即可登陆
这时候，就可以通过xshell远程登陆了


可以通过VMWARE的克隆功能，将虚拟机克隆成多个虚拟机，减少重复配置，注意，每一个虚拟机需要配置不同的ip地址



# 虚拟机上Hadoop和Java的安装

在/opt下创建两个目录，一个是software，一个是module，其中，software用来放软件包，module放解压后的软件

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gs2yw7yh6oj30sm08x78w.jpg)

更改目录的所有者，变成lowkeysp

```
sudo chown lowkeysp:lowkeysp software/ module/
```
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gs2yxprs0lj30sy05qacz.jpg)

之后，将下载好的jdk和hadoop的包放到software目录下，然后解压到module目录下


配置环境变量

在`/etc/profile.d/`目录下，创建一个`hadoop_java_env.sh`文件
```
# Java Home
export JAVA_HOME=/opt/module/jdk-11.0.11
export PATH=$PATH:$JAVA_HOME/bin


# Hadoop Home
export HADOOP_HOME=/opt/module/hadoop-3.3.1
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

```

这样，环境变量就配置完成

# 分发脚本

## 两个命令的解释

scp(secure copy)命令： 将文件从一个服务器传送到另一个服务器上，

基本语法
```
scp -r source_user@source_hostpdir/fname   dst_user@dst_host:pdir/fname
```
其中，-r表示递归，将目录下的文件都传送到目的服务器上。pdir是目录，fname是目录下的文件名称。user是目的地用户，host是主机地址。


rsync 主要用于备份和镜像，具有速度快，避免复制相同内容和支持符号链接的优点

rsync和scp的区别：用rsync做文件的复制比scp的速度快，rsync只对差异化文件做更新，而scp是把所有文件都复制过去。

基本语法：
```
rsync -av   source_user@source_hostpdir/fname   dst_user@dst_host:pdir/fname
```
-a 归档拷贝,-v显示复制过程


## SSH免密登陆

比如，如果192.168.10.1 需要免密登陆 192.168.10.2，那么操作如下：

在192.168.10.1上,执行如下命令，形成一组公钥，私钥
```
ssh-keygen -t rsa
```

将公钥发送给192.168.10.2，输入命令后，回车，得输入一个yes，才能推送成功
```
ssh-copy-id 192.168.10.2
```

这样192.168.10.1就能免密登陆 192.168.10.2了

## 具体分发脚本

shell文件名: xsync


```shell
#!/bin/bash

# 判断参数个数，如果参数个数小于1，该命令需要参数至少有1个
if [ $# -lt 1 ]
then 
    echo Not Enough Arguement!
    exit;
fi


# 遍及所有的集群

for host in 192.168.10.3 192.168.10.4
do
    echo ===============================$host========================
    
    for file in $@
    do
	# 判断文件是否存在
        if [ -e $file ]
            then
		# 获取父目录，dirname $file 可以获取父目录，pwd是显示目录，-P表示忽略软连接
		pdir=$(cd -P $(dirname $file); pwd)
		#获取当前文件的名称
		fname=$(basename $file)
		# 登陆host主机，并且创建一个目录，-p表示如果目录存在的话，则忽略,不然会报错
		ssh $host "mkdir -p $pdir"
                #将文件复制到host里面去
		rsync -av $pdir/$fname $host:$pdir
	    else
		echo $file does not exits!
	fi

    done
done

```

保存退出后，需要变成可执行文件
```
sudo chmod 775 xsync
```


之后，就可以执行分发了

比如分发hadoop和java的环境变量
```
sudo ./xsync /etc/profile.d/hadoop_java_env.sh
```


# 集群的配置


## 集群的部署规划：

* NameNode和SecondaryNameNode不要安装在同一个台服务器上
* ResourceManager也很消耗内存，不要和NameNode和SecondaryNameNode配置在同一台机器上

总结：NameNode，SecondaryNameNode，ResourceManager要部署在三台机器上。


## 配置文件说明

Hadoop配置文件分两类：默认配置文件和自定义配置文件。当用户想修改某一默认配置时，需要修改自定义配置文件，更改相应属性值

### 默认配置文件的位置：

* core-default.xml 文件放在hadoop的jar包中的 hadoop-common-3.3.1.jar/core-default.xml

* hdfs-default.xml 文件放在hadoop的jar包中的 hadoop-hdfs-3.3.1.jar/hdfs-default.xml

* yarn-default.xml 文件放在hadoop的jar包中的 hadoop-yarn-common-3.3.1.jar/yarn-default.xml

* mapred-default.xml 文件放在hadoop的jar包中的 hadoop-mapreduce-client-core-3.3.1.jar/mapred-default.xml

### 自定义配置文件的位置

core-site.xml,hdfs-site.xml,yarn-site.xml,mapred-site.xml 四个配置文件均存放在 `$HADOOP_HOME/etc/hadoop`下，用户可以根据自己的需求对其进行重新配置


### 文件的配置

假设，使用三台主机，搭建hadoop集群，地址分别为192.168.10.1，192.168.10.3，192.168.10.4。

配置策略如下：


|     |  192.168.10.1   | 192.168.10.3  | 192.168.10.4 |
|-----|  ----   | ----  | ----  | 
| HDFS| NameNode DataNode  | DataNode | SecondaryNameNode DataNode |
| HDFS| NodeManager  |  ResourceManager NodeManager |NodeManager |




* 配置core-site.xml

指定了NameNode的地址，以及Hadoop的数据存储位置
```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <!-- 指定NameNode的地址-->
    <property>
        <name>fs.defaultFS</name>
	<value>hdfs://192.168.10.1:8020</value>
    </property>
    <!-- 指定hadoop数据的存储目录 -->

    <property>
        <name>hadoop.tmp.dir</name>
	<value>/opt/module/hadoop-3.3.1/data</value>
    </property>

    

</configuration>

```

* 配置hdfs-site.xml


配置了NameNode和SecondaryNameNode的Web端访问地址

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <!--  NameNode的web端访问地址-->
    <property>
	    <name>dfs.namenode.http-address</name>
	    <value>192.168.10.1:9870</value>
    </property>

    <!-- SecondaryNameNode的Web端访问地址-->
    <property>
	    <name>dfs.namenode.secondary.http-address</name>
        <value>192.168.10.4:9868</value>
    </property>
</configuration>
```

* 配置yarn-site.xml

指定了ResourceManager为192.168.10.3

```
<?xml version="1.0"?>

<configuration>

<!-- Site specific YARN configuration properties -->
    <!-- 指定Map Reduce走shuffle-->	
    <property>
        <name>yarn.nodemanager.aux-services</name>
	    <value>mapreduce_shuffle</value>
    </property>

    <!--  指定ResourceManager 的地址-->
    <property>
	<name>yarn.resourcemanager.hostname</name>
	<value>192.168.10.3</value>
    </property>
</configuration>

```

* 配置mapred-site.xml文件

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>


<!-- Put site-specific property overrides in this file. -->

<configuration>

    <!-- 指定mapreduce程序运行在Yarn上-->
    <property>
	    <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

</configuration>

```

## 配置Workers

在`/opt/module/hadoop-3.3.1/etc/hadoop/`目录下，有一个workers文件，在workers中增加内容
```
192.168.10.1
192.168.10.3
192.168.10.4
```
注意，该文件中添加的内容结尾不允许有空格，文件中不允许有空行

之后，同步配置到所有节点
```
~/xsync workers
```

## 启动集群

1）如果集群是第一次启动，需要在192.168.10.1节点格式化NameNode（**注意：格式化NameNode，会产生新的集群ID，导致NameNode和DataNode的集群ID不一致，集群找不到以往数据。如果集群在运行过程中报错，需要重新格式化NameNode的话，一定要先停止NameNode和DataNode进程，并且要删除所有机器的data和logs目录，然后再进行格式化**）

### 格式化
```
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ hdfs namenode -format
```
格式化完成后，会发现，目录下多个两个目录,data目录和logs目录
```
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ ls
bin   etc      lib      LICENSE-binary   LICENSE.txt  NOTICE-binary  README.txt  share
data  include  libexec  licenses-binary  logs         NOTICE.txt     sbin

```

进入到data目录，
```
/opt/module/hadoop-3.3.1/data/dfs/name/current

在current目录下，有以下文件

lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1/data/dfs/name/current$ ls
fsimage_0000000000000000000  fsimage_0000000000000000000.md5  seen_txid  VERSION

其中，VERSION文件里为：
#Sat Jul 03 21:36:04 CST 2021
namespaceID=159680868
blockpoolID=BP-725394105-127.0.1.1-1625319364440
storageType=NAME_NODE
cTime=1625319364440
clusterID=CID-fb46ba01-abfa-4895-b1f0-20cd050a456c
layoutVersion=-66
```
### 格式化后，启动hdfs


启动前，还需要在`/opt/module/hadoop-3.3.1/etc/hadoop`目录下的`hadoop-env.sh`文件中添加JAVA_HOME，否则会报错

```
...

# The java implementation to use. By default, this environment
# variable is REQUIRED on ALL platforms except OS X!
export JAVA_HOME=/opt/module/jdk-11.0.11

...

```
然后分发给其他主机，之后，

在192.168.10.1主机上，启动
```
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ sbin/start-dfs.sh 
Starting namenodes on [192.168.10.1]
192.168.10.1: namenode is running as process 5414.  Stop it first and ensure /tmp/hadoop-lowkeysp-00-namenode.pid file is empty before retry.
Starting datanodes
192.168.10.3: WARNING: /opt/module/hadoop-3.3.1/logs does not exist. Creating.
192.168.10.4: WARNING: /opt/module/hadoop-3.3.1/logs does not exist. Creating.
192.168.10.1: datanode is running as process 5566.  Stop it first and ensure /tmp/hadoop-lowkeysp-00-datanode.pid file is empty before retry.
Starting secondary namenodes [192.168.10.4]

```

则启动完毕。

检查一下，

192.168.10.1上,看到有NameNode和DataNode
```
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ jps
5414 NameNode
6253 Jps
5566 DataNode

```

在192.168.10.3上，有DataNode
```
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1/etc/hadoop$ jps
3144 DataNode
3257 Jps

```

在192.168.10.4上，有SecondaryNameNode和DataNode,跟之前规划部署的都一样
```
lowkeysp-00@lowkeysp-00:~$ jps
3350 Jps
3243 SecondaryNameNode
3135 DataNode

```



我们可以通过Web页面访问，之前在xml的配置中，有配置面向web的地址，这个是Name Node的Web地址，可以用来查看HDFS上存储的数据情况

NameNode的Web地址:192.168.10.1:9870

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gs463hdyckj31ac0jjac4.jpg)

其中，首页就显示了VERSION

常用的是Utilities的Browse the file System,可以查看文件系统的情况，查看HDFS上存储的数据


### 启动Yarn

需要注意，之前部署规划时，yarn的ResourceManager是部署在192.168.10.3的，所以需要在192.168.10.3上启动yarn

```
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ sbin/start-yarn.sh 
Starting resourcemanager
resourcemanager is running as process 3370.  Stop it first and ensure /tmp/hadoop-lowkeysp-00-resourcemanager.pid file is empty before retry.
Starting nodemanagers
192.168.10.4: nodemanager is running as process 3470.  Stop it first and ensure /tmp/hadoop-lowkeysp-00-nodemanager.pid file is empty before retry.
192.168.10.1: nodemanager is running as process 6367.  Stop it first and ensure /tmp/hadoop-lowkeysp-00-nodemanager.pid file is empty before retry.

```

启动完成后，查看

192.168.10.1上,看到有NodeManager
```
5414 NameNode
6567 Jps
5566 DataNode
6367 NodeManager

```

在192.168.10.3上，有ResourceManager，NodeManager
```
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ jps
4209 Jps
3144 DataNode
4058 NodeManager
3370 ResourceManager


```

在192.168.10.4上，有NodeManager,跟之前规划部署的都一样
```
lowkeysp-00@lowkeysp-00:~$ jps
3243 SecondaryNameNode
3470 NodeManager
3822 Jps
3135 DataNode
```

Yarn也有一个web页面，之前在xml上也配置了，网址为：http://192.168.10.3:8088/   （后面的8088是固定的，xml也没有这个8088）

### 集群基本测试

这样，集群就搭建运行起来了，我们进行一个小的测试

上传数据到hdfs上
```
# 创建了一个目录，这个目录是在/目录下，这里的根目录不是服务器上实际的根目录，而是hdfs上面的根目录
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ hadoop fs -mkdir /wcinput
```
查看页面，确实也有了一个目录
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gs46ofzcq8j31do0hg0tk.jpg)

```
# 上传文件,将本地的a.txt文件上传到了/wcinput目录下
lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1$ hadoop fs -put ./aa.txt /wcinput

```

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gs46rxvdwlj31eb0ghgmj.jpg)


其实，这些数据实际存储在core-site.xml里面当时指定的hadoop数据的存储目录下，这里也就是`/opt/module/hadoop-3.3.1/data`目录下

```
/opt/module/hadoop-3.3.1/data/dfs/data/current/BP-725394105-127.0.1.1-1625319364440/current/finalized/subdir0/subdir0

这个目录下有以下文件

lowkeysp-00@lowkeysp-00:/opt/module/hadoop-3.3.1/data/dfs/data/current/BP-725394105-127.0.1.1-1625319364440/current/finalized/subdir0/subdir0$ ll
总用量 16
drwxrwxr-x 2 lowkeysp-00 lowkeysp-00 4096 7月   3 23:37 ./
drwxrwxr-x 3 lowkeysp-00 lowkeysp-00 4096 7月   3 23:37 ../
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00   16 7月   3 23:37 blk_1073741825
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00   11 7月   3 23:37 blk_1073741825_1001.meta


cat blk_1073741825
a
aa
aaaaa
aa
a


所以这个文件就是之前的aa.txt
```

而且，上传文件后，不光是上传到了192.168.10.1上，而是存储到了三台（Replication=3）DataNode上

执行wordcount

```
lowkeysp-00@lowkeysp-01:/opt/module/hadoop-3.3.1$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.1.jar wordcount /wcinput /wcoutput
2021-07-04 20:30:04,464 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at /192.168.10.3:8032
2021-07-04 20:30:04,875 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/lowkeysp-00/.staging/job_1625401657661_0001
2021-07-04 20:30:05,214 INFO input.FileInputFormat: Total input files to process : 1
2021-07-04 20:30:05,330 INFO mapreduce.JobSubmitter: number of splits:1
2021-07-04 20:30:05,618 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1625401657661_0001
2021-07-04 20:30:05,618 INFO mapreduce.JobSubmitter: Executing with tokens: []
2021-07-04 20:30:05,793 INFO conf.Configuration: resource-types.xml not found
2021-07-04 20:30:05,793 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2021-07-04 20:30:06,184 INFO impl.YarnClientImpl: Submitted application application_1625401657661_0001
2021-07-04 20:30:06,229 INFO mapreduce.Job: The url to track the job: http://lowkeysp-03:8088/proxy/application_1625401657661_0001/
2021-07-04 20:30:06,230 INFO mapreduce.Job: Running job: job_1625401657661_0001
2021-07-04 20:30:13,461 INFO mapreduce.Job: Job job_1625401657661_0001 running in uber mode : false
2021-07-04 20:30:13,463 INFO mapreduce.Job:  map 0% reduce 0%
2021-07-04 20:30:20,540 INFO mapreduce.Job:  map 100% reduce 0%
2021-07-04 20:30:25,573 INFO mapreduce.Job:  map 100% reduce 100%
2021-07-04 20:30:26,586 INFO mapreduce.Job: Job job_1625401657661_0001 completed successfully
2021-07-04 20:30:26,660 INFO mapreduce.Job: Counters: 54
	File System Counters
		FILE: Number of bytes read=35
		FILE: Number of bytes written=544757
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=120
		HDFS: Number of bytes written=17
		HDFS: Number of read operations=8
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
		HDFS: Number of bytes read erasure-coded=0
	Job Counters 
		Launched map tasks=1
		Launched reduce tasks=1
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=3893
		Total time spent by all reduces in occupied slots (ms)=2872
		Total time spent by all map tasks (ms)=3893
		Total time spent by all reduce tasks (ms)=2872
		Total vcore-milliseconds taken by all map tasks=3893
		Total vcore-milliseconds taken by all reduce tasks=2872
		Total megabyte-milliseconds taken by all map tasks=3986432
		Total megabyte-milliseconds taken by all reduce tasks=2940928
	Map-Reduce Framework
		Map input records=5
		Map output records=5
		Map output bytes=36
		Map output materialized bytes=35
		Input split bytes=104
		Combine input records=5
		Combine output records=3
		Reduce input groups=3
		Reduce shuffle bytes=35
		Reduce input records=3
		Reduce output records=3
		Spilled Records=6
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=61
		CPU time spent (ms)=1670
		Physical memory (bytes) snapshot=520179712
		Virtual memory (bytes) snapshot=5475196928
		Total committed heap usage (bytes)=325058560
		Peak Map Physical memory (bytes)=305750016
		Peak Map Virtual memory (bytes)=2729951232
		Peak Reduce Physical memory (bytes)=214429696
		Peak Reduce Virtual memory (bytes)=2745245696
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=16
	File Output Format Counters 
		Bytes Written=17
```
则，计算成功

在相应的Web网页上也能看到相应的信息和结果


## 集群崩溃异常如何处理

处理方法：格式化NameNode

第一步：先停掉进程
```
sbin/stop-dfs.sh
sbin/stop-yarn.sh
```

第二步，删掉data/和logs/,所有的机器都要删掉
```
rm -rf data/ logs/
```


第三步，格式化
```
hdfs namenode -format
```

## 配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：


配置 etc/hadoop/mapred-site.xml
```
    <!-- 历史服务器地址，对内-->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>192.168.10.1:10020</value>
    </property>

    <!--  历史服务器Web地址，对外-->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>192.168.10.1:19888</value>
    </property>

```

分发配置给其他服务器
```
~/xsync etc/hadoop/mapred-site.xml 
```

关闭yarn进程

在192.168.10.1上启动历史服务器
```
bin/mapred --daemon start historyserver


启动之后，查看确实多了一个jobhistoryserver

lowkeysp-00@lowkeysp-01:/opt/module/hadoop-3.3.1$ jps
2161 NameNode
2311 DataNode
23528 Jps
23146 JobHistoryServer
18335 NodeManager

```


启动历史服务器之后，就可以在网页上查看
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gs57w8gqxcj61c00k0tas02.jpg)

## 配置日志的聚集

日志聚集概念：应用运行完成后，将程序运行日志信息上传到HDFS上。从而可以方便查看到程序的运行详情，方便开发调试

注意：开启日志聚集功能，需要重新启动NodeManager,ResourceManager和HistoryServer

配置yarn-site.xml

```
    <!--开启日志聚集功能    -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>

    <!-- 设置日志聚集服务器的地址-->
    <property>
        <name>yarn.log.server.url</name>
        <value>http://192.168.10.1:19888/jobhistory/logs</value>
    </property>

    <!-- 设置日志保留时间为7天-->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>

```

分发配置
```
 ~/xsync etc/hadoop/yarn-site.xm
```

关闭NodeManager,ResourceManager和HistoryServer
```
sbin/stop-yarn.sh

mapred --daemon stop historyserver
```


再打开
```
sbin/start-yarn.sh

mapred --daemon start historyserver
```

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gs5b3xq0axj31es0krwgu.jpg)



## 逐一开启/停止hdfs和yarn的组件

```
# 分别开启/停止namenode/datanode/secondarynamenode
hdfs --daemon start/stop namenode/datanode/secondarynamenode

# 
# 分别开启/停止resourcemanager/nodemanager
yarn --daemon start/stop resourcemanager/nodemanager
```

## 开启集群的脚本和查看进程的脚本

```

#!/bin/bash

if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit;
fi


case $1 in
"start")
    echo "================================启动Hadoop集群===================="

    echo "================================启动HDFS=========================="
    ssh 192.168.10.1 "/opt/module/hadoop-3.3.1/sbin/start-dfs.sh"
    echo "================================启动Yarn=========================="
    ssh 192.168.10.3 "/opt/module/hadoop-3.3.1/sbin/start-yarn.sh"
    echo "================================启动historyserver=========================="
    ssh 192.168.10.1 "/opt/module/hadoop-3.3.1/bin/mapred --daemon start historyserver"

;;
"stop")
    echo "================================关闭Hadoop集群===================="

    echo "================================关闭historyserver=========================="
    ssh 192.168.10.1 "/opt/module/hadoop-3.3.1/bin/mapred --daemon stop historyserver"
    echo "===============================关闭Yarn=========================="
    ssh 192.168.10.3 "/opt/module/hadoop-3.3.1/sbin/stop-yarn.sh"
    echo "================================关闭HDFS=========================="
    ssh 192.168.10.1 "/opt/module/hadoop-3.3.1/sbin/stop-dfs.sh"
;;
esac
```

查看各节点进程
```
#!/bin/bash
  
for host in 192.168.10.1 192.168.10.3 192.168.10.4
do
    echo ========================$host========================
    ssh $host "/opt/module/jdk-11.0.11/bin/jps"
done

```


## 常用端口号
hadoop3.X
> * HDFS NameNode 内部通信端口：8020/9000/9820
> * HDFS NameNode 对外的查询端口：9870
> * Yarn查看任务情况的端口：8088
> * 历史服务器：19888

hadoop2.X
> * HDFS NameNode 内部通信端口：8020/9000
> * HDFS NameNode 对外的查询端口：50070
> * Yarn查看任务情况的端口：8088
> * 历史服务器：19888


## 服务器时间同步

如果服务器能与外网连接，则不需要时间同步，否则，需要时间同步


