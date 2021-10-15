---
title: Yarn学习
date: 2021-10-13 11:07:49
tags:
---

<meta name="referrer" content="no-referrer" />

# Yarn的基本架构
由Resource Manager，Node Manager，Application Master，Container组成

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gvdilz0e48j61h30r5at402.jpg)

# Yarn调度器

## FIFO调度器
FIFO调度器将应用放置在一个队列中，然后按照提交的顺序（先进先出）运行应用。首先为队列中第一个应用的请求分配资源，第一个应用的请求被满足后再依次为队列中下一个应用服务。

FIFO调度器的优点是：简单易懂，不需要任何配置。但是不适合共享集群。大的应用会占用集群中的所有资源，所以每个应用必须等待直到轮到自己运行。在一个共享集群中，更适合容量调度器和公平调度器。这两种调度器都允许长时间运行的作业能够及时完成，同时也允许正在进行较小临时查询的用户能够在合理时间内得到返回结果。

## 容量调度器
Hadoop默认是容量调度器

容量调度器允许多个组织共享一个hadoop集群，每个组织可以分配到全部集群资源的一部分。每个组织被配置一个专门的队列，每个队列被配置为可以使用一定的集群资源。队列还可以进一步按照层次划分，这样每个组织内的不同用户能供共享该组织队列所分配的资源。在一个队列里，使用FIFO调度策略进行调度。

单个作业所使用的资源不会超过其队列容量。然而，如果一个队列中有多个作业，并且队列资源不够的话，如果仍有可用的空闲资源，那么容量调度器可能会将空余的资源分配给队列中的作业，哪怕这会超出队列容量，这称为“弹性队列”

正常操作时，容量调度器不会通过强行中止来抢占容器，因此，如果一个队列一开始资源够用，然后随着需求增长，资源开始不够用时，那么这个队列就只能等着其他队列释放容器资源。缓解这种情况的方法是：对队列设置一个最大容量限制，这样这个队列就不会过多侵占其他队列的容量了。

## 公平调度器

公平调度器旨在为所有运行的应用公平分配资源。包括同一个队列中的应用实现资源公平共享，多个队列间实现公平共享。

如何在队列之间公平共享？想象两个用户A和B，分别拥有自己的队列。A启动一个作业，在B没有需求时，A会分配到全部可用资源；当A的作业仍在运行时B启动一个作业，一段时间后，每个作业都用到了一半的集群资源。这时，如果B启动第二个作业且其他作业仍在运行，那么第二个作业将和B的其他作业（这里是第一个）共享资源，因此B的每个作业将占用四分之一的集群资源，而A仍继续占用一半的集群资源。


# Yarn常用命令
* 展示运行中任务的详细信息
`yarn application-list`

* 筛选出某个状态的任务
`yarn application-list-appStates` 

* kill某一个任务
`yarn application -kill <applicationId>`

* 查看applicationd的logs
`yarn logs-applicationId <applicationId>`

* 查看Container日志
`yarn logs-applicationId <applicationId> -containerId <containerId>`

* 查看尝试运行的任务
`yarn applicationattempt -list <applicationId>`

* 查看application所用的容器
`yarn container -list <applicationattemptId>`

* 打印Container状态
`yarn container -status <ContainerId>`

* 列出所有节点
`yarn node -list all`

* yarn rmadmin 刷新配置,在运行时如果对队列配置文件有更改时，可用这个命令刷新配置
`yarn rmadmin -refreshQueues`

* yarn queue状态
`yarn queue -status default`

# Yarn生产环境核心配置参数

## ResourceManager

* `yarn.resourcemanager.scheduler.class`：配置调度器，默认容量调度器
* `yarn.resourcemanager.scheduler.client.thread-count`：ResourceManager处理调度器请求的线程数量，默认50

## NodeManager
* `yarn.nodemanager.resource.detect-hardware-capabilities`：是否让yarn自己检测硬件进行配置，默认false
* `yarn.nodemanager.resource.count-logical-processors-as-cores`：是否将虚拟核数当作cpu核数，默认false
* `yarn.nodemanager.resource.pcores-vcores-multiplier`：虚拟核数和物理核数乘数，例如：4核8线程，该参数应该设置为2，默认为1
* `yarn.nodemanager.resource.memory-mb`：NodeManager使用内存，默认8G
* `yarn.nodemanager.resource.system-reserved-memory-mb`：NodeManager为系统保留多少内存，跟上面的参数配置一个即可
* `yarn.nodemanager.resource.cpu-vcores`：NodeManager使用CPU核数，默认8个
* `yarn.nodemanager.resource.pmem-check-enabled`：是否开启物理内存检查限制container，默认打开
* `yarn.nodemanager.resource.vmem-check-enabled`：是否开启虚拟内存检查限制container，默认打开
* `yarn.nodemanager.resource.vmem-pmem-ratio`：虚拟内存物理内存比例，默认2.1

## Container

* `yarn.scheduler.minimum-allocation-mb`：容器最小内存，默认1G
* `yarn.scheduler.maximum-allocation-mb`: 容器最大内存，默认8G
* `yarn.scheduler.minimum-allocation-vcores`: 容器最小CPU核数，默认1个
* `yarn.scheduler.maximum-allocation-vcores`：容器默认最大CPU核数，默认4个

## 配置实例

在yarn-site.xml文件中
```xml
<!-- 选择调度器，默认容量调度器，中小型用容量调度器，大型用公平调度器 -->

<property>
    <description>The class to use as the resource scheduler.</description>
    <name>yarn.resourcemanager.scheduler.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>

<!--ResourceManager处理调度器请求的线程数量，默认50；通常不能超过 N * M ，其中N为机器数量，M为线程数量，比如3台机器数量，4个线程数量，则不能超过12，默认不要超过8，因为机器还可能运行其他程序  -->

<property>
    <description>Number of threads to handle scheduler interface.</description>
    <name>yarn.resourcemanager.scheduler.client.thread-count</name>
    <value>50</value>
</property>


<!-- 是否让yarn自动检测硬件进行配置，默认是false。如果该节点有很多其他应用程序，建议手动配置，如果没有其他应用程序，可以采用自动 -->

  <property>
    <description>Enable auto-detection of node capabilities such as
    memory and CPU.
    </description>
    <name>yarn.nodemanager.resource.detect-hardware-capabilities</name>
    <value>false</value>
  </property>


<!-- 是否将虚拟核数当作cpu核数，默认false -->

  <property>
    <description>Flag to determine if logical processors(such as
    hyperthreads) should be counted as cores. Only applicable on Linux
    when yarn.nodemanager.resource.cpu-vcores is set to -1 and
    yarn.nodemanager.resource.detect-hardware-capabilities is true.
    </description>
    <name>yarn.nodemanager.resource.count-logical-processors-as-cores</name>
    <value>false</value>
  </property>


<!-- 虚拟核数和物理核数乘数，例如：4核8线程，默认为1 -->
  <property>
    <description>Multiplier to determine how to convert phyiscal cores to
    vcores. This value is used if yarn.nodemanager.resource.cpu-vcores
    is set to -1(which implies auto-calculate vcores) and
    yarn.nodemanager.resource.detect-hardware-capabilities is set to true. The
    number of vcores will be calculated as
    number of CPUs * multiplier.
    </description>
    <name>yarn.nodemanager.resource.pcores-vcores-multiplier</name>
    <value>1.0</value>
  </property>


<!-- NodeManager使用内存,需要根据机器内存进行调整，默认是8G，即8192MB-->
   <property>
    <description>Amount of physical memory, in MB, that can be allocated 
    for containers. If set to -1 and
    yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
    automatically calculated(in case of Windows and Linux).
    In other cases, the default is 8192MB.
    </description>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>-1</value>
  </property>


<!-- NodeManager使用CPU核数，默认8个 -->

 <property>
    <description>Number of vcores that can be allocated
    for containers. This is used by the RM scheduler when allocating
    resources for containers. This is not used to limit the number of
    CPUs used by YARN containers. If it is set to -1 and
    yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
    automatically determined from the hardware in case of Windows and Linux.
    In other cases, number of vcores is 8 by default.</description>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>-1</value>
  </property>


<!-- 容器最小内存，默认1G-->
  <property>
    <description>The minimum allocation for every container request at the RM
    in MBs. Memory requests lower than this will be set to the value of this
    property. Additionally, a node manager that is configured to have less memory
    than this value will be shut down by the resource manager.</description>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>1024</value>
  </property>

<!-- 容器最大内存，默认8G  -->
  <property>
    <description>The maximum allocation for every container request at the RM
    in MBs. Memory requests higher than this will throw an
    InvalidResourceRequestException.</description>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>8192</value>
  </property>



<!-- 容器最小CPU核数，默认1个 -->
  <property>
    <description>The minimum allocation for every container request at the RM
    in terms of virtual CPU cores. Requests lower than this will be set to the
    value of this property. Additionally, a node manager that is configured to
    have fewer virtual cores than this value will be shut down by the resource
    manager.</description>
    <name>yarn.scheduler.minimum-allocation-vcores</name>
    <value>1</value>
  </property>


<!-- 容器默认最大CPU核数，默认4个 -->
  <property>
    <description>The maximum allocation for every container request at the RM
    in terms of virtual CPU cores. Requests higher than this will throw an
    InvalidResourceRequestException.</description>
    <name>yarn.scheduler.maximum-allocation-vcores</name>
    <value>4</value>
  </property>
```

# 配置容量调度器的队列以及优先级

需求：假如要有两个队列，default和hive队列。其中，default队列占总内存40%，最大资源容量占总资源60%，hive队列占总内存的60%，最大资源容量占总资源80%。并且，hive的优先级比default队列的优先级高

在capacity-scheduler.xml中配置

```xml

<!-- 指定多队列，增加hive队列 -->
  <property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
  </property>



<!--  配置default队列为40，hive队列为60-->
  <property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>40</value>
    <description>Default queue target capacity.</description>
  </property>

    <property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
    <value>60</value>
    <description>Default queue target capacity.</description>
  </property>

<!--  向该队列提交任务时，能占用这个队列资源的百分比，比如为1，则可以全部占用，如0.5，则用户只能占用这个队列的50%-->

 <property>
    <name>yarn.scheduler.capacity.root.default.user-limit-factor</name>
    <value>1</value>
    <description>
      Default queue user limit a percentage from 0.0 to 1.0.
    </description>
  </property>

 <property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
    <value>1</value>
    <description>
      Default queue user limit a percentage from 0.0 to 1.0.
    </description>
  </property>



<!-- 队列最大可以占用的资源容量，default是60%，hive是80%-->
  <property>
    <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
    <value>60</value>
    <description>
      The maximum capacity of the default queue. 
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
    <value>80</value>
    <description>
      The maximum capacity of the default queue. 
    </description>
  </property>


<!-- 要将队列设置成运行状态-->
  <property>
    <name>yarn.scheduler.capacity.root.default.state</name>
    <value>RUNNING</value>
    <description>
      The state of the default queue. State can be one of RUNNING or STOPPED.
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
    <description>
      The state of the default queue. State can be one of RUNNING or STOPPED.
    </description>
  </property>

<!-- 设置哪些用户可以提交任务到队列，*表示所有用户，-->
 <property>
    <name>yarn.scheduler.capacity.root.default.acl_submit_applications</name>
    <value>*</value>
    <description>
      The ACL of who can submit jobs to the default queue.
    </description>
  </property>

 <property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
    <value>*</value>
    <description>
      The ACL of who can submit jobs to the default queue.
    </description>
  </property>


<!-- 对队列的操作权限-->
  <property>
    <name>yarn.scheduler.capacity.root.default.acl_administer_queue</name>
    <value>*</value>
    <description>
      The ACL of who can administer jobs on the default queue.
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
    <value>*</value>
    <description>
      The ACL of who can administer jobs on the default queue.
    </description>
  </property>




<!-- 配置优先级-->
  <property>
    <name>yarn.scheduler.capacity.root.default.acl_application_max_priority</name>
    <value>*</value>
    <description>
      The ACL of who can submit applications with configured priority.
      For e.g, [user={name} group={name} max_priority={priority} default_priority={priority}]
    </description>
  </property>



<!-- 任务最大的生存周期，如果超过这个时间，则会kill掉这个任务-->

   <property>
     <name>yarn.scheduler.capacity.root.default.maximum-application-lifetime
     </name>
     <value>-1</value>
     <description>
        Maximum lifetime of an application which is submitted to a queue
        in seconds. Any value less than or equal to zero will be considered as
        disabled.
        This will be a hard time limit for all applications in this
        queue. If positive value is configured then any application submitted
        to this queue will be killed after exceeds the configured lifetime.
        User can also specify lifetime per application basis in
        application submission context. But user lifetime will be
        overridden if it exceeds queue maximum lifetime. It is point-in-time
        configuration.
        Note : Configuring too low value will result in killing application
        sooner. This feature is applicable only for leaf queue.
     </description>
   </property>

   <property>
     <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime
     </name>
     <value>-1</value>
     <description>
        Maximum lifetime of an application which is submitted to a queue
        in seconds. Any value less than or equal to zero will be considered as
        disabled.
        This will be a hard time limit for all applications in this
        queue. If positive value is configured then any application submitted
        to this queue will be killed after exceeds the configured lifetime.
        User can also specify lifetime per application basis in
        application submission context. But user lifetime will be
        overridden if it exceeds queue maximum lifetime. It is point-in-time
        configuration.
        Note : Configuring too low value will result in killing application
        sooner. This feature is applicable only for leaf queue.
     </description>
   </property>


<!-- 任务默认的生命周期 -->
   <property>
     <name>yarn.scheduler.capacity.root.default.default-application-lifetime
     </name>
     <value>-1</value>
     <description>
        Default lifetime of an application which is submitted to a queue
        in seconds. Any value less than or equal to zero will be considered as
        disabled.
        If the user has not submitted application with lifetime value then this
        value will be taken. It is point-in-time configuration.
        Note : Default lifetime can't exceed maximum lifetime. This feature is
        applicable only for leaf queue.
     </description>
   </property>

   <property>
     <name>yarn.scheduler.capacity.root.hive.default-application-lifetime
     </name>
     <value>-1</value>
     <description>
        Default lifetime of an application which is submitted to a queue
        in seconds. Any value less than or equal to zero will be considered as
        disabled.
        If the user has not submitted application with lifetime value then this
        value will be taken. It is point-in-time configuration.
        Note : Default lifetime can't exceed maximum lifetime. This feature is
        applicable only for leaf queue.
     </description>
   </property>
```

# 任务优先级

修改yarn-site.xml文件
```xml

<!-- 表示设置了5个优先级 -->
<property>
    <name>yarn.cluster.max-application-priority</name>
    <value>5</value>
</property>

```

则再提交新任务时，可以明确优先级
```shell

hadoop jar jar包 wordcount -D mapreduce.job.priority=5 ... ...

```

如果修改正在运行的任务的优先级，可使用下面的命令：
```shell
yarn application -appID appID -updatePriority 5
```

# 公平调度器配置

配置涉及两个文件，yarn-site.xml和fair-scheduler.xml

首先，需要配置yarn-site.xml
```xml
<!-- 设置要使用公平调度器 -->
<property>
  <name>yarn.resourcemanager.scheduler.class</name>
  <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
</property>

<!-- 指明公平调度器队列所使用的配置文件位置-->
<property>
  <name>yarn.scheduler.fair.allocation.file</name>
  <value>/opt/module/hadoop-3.3.1/etc/hadoop/fair-scheduler.xml</value>
</property>

<!-- 是否要开启队列间资源抢占，默认false-->
<property>
  <name>yarn.scheduler.fair.preemption</name>
  <value>false</value>
</property>

```


配置fair-scheduler.xml
```xml
<?xml version="1.0"?>
<allocations>

  <!-- 单个队列中Application Master占用资源的最大比例，取值0-1，企业一般配置0.1-->
  <queueMaxAMShareDefault>0.1</queueMaxAMShareDefault>
  <!--单个队列最大资源的默认值 -->
  <queueMaxResourcesDefault>4096mb,4vcores</queueMaxResourcesDefault>

    <!-- 增加一个队列-->
    <queue name="production">
        <!--  队列最小资源 -->
        <minResources>1024 mb, 10 vcores</minResources>
        <!-- 队列最大资源 -->
        <maxResources>5120 mb, 20 vcores</maxResources>
        <!-- 队列中同时运行的应用数，默认50，根据线程数配置-->
        <maxRunningApps>50</maxRunningApps>
        <!-- 队列中Application Master占用资源的最大比例-->
        <maxAMShare>0.5</maxAMShare>
        <!-- 该队列资源权重，默认为1-->
        <weight>1.0</weight>
        <!-- 队列内部的资源分配策略，“fifo”/“fair”/“drf” -->
        <schedulingPolicy>fair</schedulingPolicy>

    </queue>
    <queue name="development">
        <weight>1.0</weight>
        <schedulingPolicy>drf</schedulingPolicy>
        <aclSubmitApps>*</aclSubmitApps>
        <aclAdministerApps>*</aclAdministerApps>
    </queue>
    <!-- 默认队列的策略，如果上面没有指定的话，则会设置成这个策略-->
    <defaultQueueSchedulingPolicy>fair</defaultQueueSchedulingPolicy>

    <!-- 告诉调度器进来的任务应该如何分配到不同的队列中 -->
    <queuePlacementPolicy>
        <!-- 如果create设置为true，则当任务指定的队列不存在时，会自动创建一个队列，默认是true，一般可设置成false-->
        <rule name="specified" create="true"/>

        <!-- 比如提交到root.group.username队列，如果root.group不存在，不允许自动创建，如果root.group.username不存在，允许自动创建-->
        <rule name="nestedUserQueue" create="true"/>
        <rule name="primaryGroup" create="false"/>
        <!-- 最后一个规则必须为reject或者default，Reject表示拒绝，default表示会把任务提交到default队列中-->
        <rule name="reject">
    </queuePlacementPolicy>
</allocations>

```
提交任务的命令：
```
hadoop jar jar包 类名 -D  mapreduce.job.queuename=root.development
```


# Tool接口
使用传统的driver代码创建mapreduce任务时，配置一般都是写死的。如果你需要更改一些配置的话，还需要更改代码，重新生成jar包，重新部署等。而如果使用Tool接口则可以避免这些，下面我们来看如何使用

下面通过不使用Tool接口和使用Tool接口进行比较

不使用Tool接口
```java
public class ToolMapReduce {
 
    public static void main(String[] args) throws Exception {
 
        // Create configuration
        Configuration conf = new Configuration();
 
        // Create job
        Job job = new Job(conf, "Tool Job");
        job.setJarByClass(ToolMapReduce.class);
 
        // Setup MapReduce job
        job.setMapperClass(Mapper.class);
        job.setReducerClass(Reducer.class);
 
        // Set only 1 reduce task
        job.setNumReduceTasks(1);
 
        // Specify key / value
        job.setOutputKeyClass(LongWritable.class);
        job.setOutputValueClass(Text.class);
 
        // Input
        FileInputFormat.addInputPath(job, new Path(args[0]));
        job.setInputFormatClass(TextInputFormat.class);
 
        // Output
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        job.setOutputFormatClass(TextOutputFormat.class);
 
        // Execute job
        int code = job.waitForCompletion(true) ? 0 : 1;
        System.exit(code);
    }
}
```
则上面的mapreduce任务需要通过下面命令执行，只能使用两个参数，inputPath和outputPath，在代码中分别位于index[0]和index[1],
```shell
hadoop jar /path/to/My/jar.jar com.wordpress.hadoopi.ToolMapReduce /input/path /output/path
```
在这种情况下，reducers的数量是写死的，在代码中写死的，只能是1，不能通过命令行更改

而如果我们使用Tool接口
```java
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
 
public class ToolMapReduce extends Configured implements Tool {
 
    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new Configuration(), new ToolMapReduce(), args);
        System.exit(res);
    }
 
    @Override
    public int run(String[] args) throws Exception {
 
        // When implementing tool
        Configuration conf = this.getConf();
 
        // Create job
        Job job = new Job(conf, "Tool Job");
        job.setJarByClass(ToolMapReduce.class);
 
        // Setup MapReduce job
        // Do not specify the number of Reducer
        job.setMapperClass(Mapper.class);
        job.setReducerClass(Reducer.class);
 
        // Specify key / value
        job.setOutputKeyClass(LongWritable.class);
        job.setOutputValueClass(Text.class);
 
        // Input
        FileInputFormat.addInputPath(job, new Path(args[0]));
        job.setInputFormatClass(TextInputFormat.class);
 
        // Output
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        job.setOutputFormatClass(TextOutputFormat.class);
 
        // Execute job and return status
        return job.waitForCompletion(true) ? 0 : 1;
    }
}
```
ToolsRunner通过它的静态的run方法执行mapreduce任务。在这个例子里，可以通过使用"-D"选项指定reducer的数量
```
hadoop jar /path/to/My/jar.jar com.wordpress.hadoopi.ToolMapReduce -D mapred.reduce.tasks=1 /input/path /output/path
```

GenericOptionParser可以将generic Tool option和真实的任务参数区分开，不管提供多少个generic option，inputPath和outputPath变量仍然在index[0]和index[1](指的是在run方法的args数组中)，其中，ToolRunner方法中其实使用了GenericOptionParser。

-D 后面可以配置各种配置，
比如，`conf.set("my.dummy.configuration","foobar")`，变成`-D my.dummy.configuration=foobar`。

当想把一个jar文件提交到一个远程的hadoop server时，需要在driver代码中指定以下配置
```java
Configuration conf = new Configuration();
conf.set("mapred.job.tracker", "myserver.com:8021");
conf.set("fs.default.name", "hdfs://myserver.com:8020");
```
如果用tool接口，则需要借助`-fs`和`-jt`，如下：
```
hadoop jar myjar.jar com.wordpress.hadoopi.ToolMapReduce -fs hdfs://myserver.com:8020 -jt myserver.com:8021
```

Options that are supported by ToolRunner through GenericOptionsParser are as follows-
```
-conf <configuration file>– Specify an application configuration file. So you can prepare an XML file and set it using -conf option that way you can set many properties at once.

-D <property>=<value>– Sets value for given property. Specifying a property with -D option will override any property with the same name in the configuration file or with in the driver code.

-fs <file:///> or <hdfs://namenode:port>– This generic option is used to specify default filesystem URL to use. Overrides ‘fs.defaultFS’ property from configurations.

-jt <local> or <resourcemanager:port>– Used to set YARN ResourceManager.

-files <comma separated list of files>– Specify comma separated files to be copied to the map reduce cluster. Applies only to job. If you want to add a file to distributed cache then rather than hardcoding it with in your driver code by using job.addCacheFile() method you can specify it using -files generic option.

-libjars <comma seperated list of jars>– Specify comma separated jar files to include in the classpath. Applies only to job. If you want to add a jar to distributed cache then rather than hardcoding it with in your driver by using job.addFileToClassPath() method you can specify it using -libjars generic option.

-archives <comma separated list of archives>– Specify comma separated archives to be unarchived on the compute machines. Applies only to job. If you want to add an archived file (zip, tar and tgz/tar.gz files) then rather than hard coding it with in your driver by using job.addCacheArchive() method you can specify it using -libjars generic option.
```



