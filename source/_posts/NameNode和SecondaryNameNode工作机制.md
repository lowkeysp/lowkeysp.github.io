---
title: NameNode，SecondaryNameNode，DataNode工作机制
date: 2021-07-14 14:27:52
tags:
---


# NameNode，SecondaryNameNode工作机制

NameNode的数据文件如下，目录为`/opt/module/hadoop-3.3.1/data/dfs/name/current`
```
lowkeysp-00@lowkeysp-01:/opt/module/hadoop-3.3.1/data/dfs/name/current$ ll
总用量 4180
drwx------ 2 lowkeysp-00 lowkeysp-00    4096 7月   4 23:18 ./
drwxrwxr-x 3 lowkeysp-00 lowkeysp-00    4096 7月   4 23:37 ../
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      42 7月   3 23:07 edits_0000000000000000001-0000000000000000002
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00    7056 7月   4 00:07 edits_0000000000000000003-0000000000000000066
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00 1048576 7月   4 00:20 edits_0000000000000000067-0000000000000000117
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00 1048576 7月   4 20:12 edits_0000000000000000118-0000000000000000169
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00   23867 7月   4 21:26 edits_0000000000000000170-0000000000000000340
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      42 7月   4 22:26 edits_0000000000000000341-0000000000000000342
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00 1048576 7月   4 22:53 edits_0000000000000000343-0000000000000000440
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      42 7月   4 23:18 edits_0000000000000000441-0000000000000000442
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00 1048576 7月   4 23:18 edits_inprogress_0000000000000000443
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00    4771 7月   4 22:26 fsimage_0000000000000000342
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      62 7月   4 22:26 fsimage_0000000000000000342.md5
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00    5702 7月   4 23:18 fsimage_0000000000000000442
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      62 7月   4 23:18 fsimage_0000000000000000442.md5
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00       4 7月   4 23:18 seen_txid
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00     212 7月   4 19:48 VERSION


```

SecondaryNameNode的数据文件如下,目录为:`/opt/module/hadoop-3.3.1/data/dfs/namesecondary/current`
```
lowkeysp-00@lowkeysp-04:/opt/module/hadoop-3.3.1/data/dfs/namesecondary/current$ ll
总用量 2128
drwxrwxr-x 2 lowkeysp-00 lowkeysp-00    4096 7月   4 23:18 ./
drwxrwxr-x 3 lowkeysp-00 lowkeysp-00    4096 7月   4 23:37 ../
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      42 7月   3 23:07 edits_0000000000000000001-0000000000000000002
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00    7056 7月   4 00:07 edits_0000000000000000003-0000000000000000066
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00 1048576 7月   4 21:26 edits_0000000000000000118-0000000000000000169
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00   23867 7月   4 21:26 edits_0000000000000000170-0000000000000000340
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      42 7月   4 22:26 edits_0000000000000000341-0000000000000000342
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00 1048576 7月   4 23:18 edits_0000000000000000343-0000000000000000440
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      42 7月   4 23:18 edits_0000000000000000441-0000000000000000442
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00    4771 7月   4 23:18 fsimage_0000000000000000342
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      62 7月   4 23:18 fsimage_0000000000000000342.md5
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00    5702 7月   4 23:18 fsimage_0000000000000000442
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00      62 7月   4 23:18 fsimage_0000000000000000442.md5
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00     212 7月   4 23:18 VERSION
```

可以看到，NameNode有如下文件：`edits_`,`edits_inprogress_`,`fsimage_`,SecondaryNameNode有如下文件：`edits_`,`fsimage_`

NameNode的元数据存储在内存中，为了防止断电导致内存中的数据丢失，在磁盘中备份了元数据，叫做`fsimage`。当内存中的元数据更新时，如果同时更新fsimage，会导致效率低下，如果不更新，则会发生不一致问题。为解决这一问题，引入了`Edits`文件。该文件只进行追加操作，效率很高。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到Edits文件中，这样，即使断电，仍可通过Fsimage和Edits合并，合成元数据

具体工作机制如下：

* 1）第一次启动NameNode格式化后，会创建`Fsimage`文件和`seen_txid`文件，其中Fsimage是HDFS系统的元数据的一个永久性的检查点，记录着整个文件目录结构以及文件每个数据块所在的节点位置，seen_txid文件记录了正在执行的事务id，初始化时为0。
* 2）当进行文件操作后，NameNode会生成`edits_`和`edits_inprogress_`文件，该文件存放了HDFS文件系统的所有更新操作。HDFS每个操作都会产生一个事务id，`edits_0000000000000000001-0000000000000000002`表示这里面记录了两个事务操作，`edits_inprogress_0000000000000000443`表示这个文件正在被编辑，且下一个事务id应该从443开始，此时seen_txid的内容如下：
```
lowkeysp-00@lowkeysp-01:/opt/module/hadoop-3.3.1/data/dfs/name/current$ cat seen_txid 
443
```
* 3)当每次NameNode启动时，内存会加载Fsimage和edits文件，保证内存中的元数据是最新的。比如将fsimage_0000000000000000442和edits_inprogress_0000000000000000443进行合并（edits_inprogress_0000000000000000443需先变成edits_00000XX，fsimage_0000000000000000442表示已经合并到442了，不会再跟1-442的edits进行合并了），然后跟发当对HDFS文件系统进行操作时，NameNode并不会直接更新fsimage，而是把该次操作先记录到edits文件中，然后再更新内存中的元数据。

* 4）SecondaryNameNode会去协助NameNode合并Fsimage和edits，而不用等到每次NameNode启动时再合并，CheckPoint的触发条件有两个：
1、SecondaryNameNode会定期询问NameNode是否需要合并，默认定时时间是1小时；2、NameNode中的edits数据满了，默认是100万条操作

* 5）如果NameNode同意合并，则NameNode滚动正在写的edits日志，即将edits_inprogress_文件变成edits_000000XX文件，比如，edits_inprogress_0000000000000000443变成edits_0000000000000000443，之后再生成edits_inprogress_0000000000000000444用于接受合并过程中对HDFS文件系统的操作。

* 6）将edits_文件和fsimage文件拷贝到SecondaryNameNode，SecondaryNameNode加载到内存中进行合并，生成fsimage.chkpoint。再将fsimage.chkpoint送回NameNode，NameNode将fsimage.chkpoint重新命名为fsimage


一小时合并一次文件，hdfs-default.xml
```
<property>
    <name>dfs.namenode.checkpoint.period</name>
    <value>3600s</value>
</property
```

操作100万次就认为满了
```

<property> 
    <name>dfs.namenode.checkpoint.txns</name> 
    <value>1000000</value>
    <description>操作动作次数</description>
 
</property>
 
<property>
    <name>dfs.namenode.checkpoint.check.period</name>
    <value>60s</value>
    <description> 1分钟检查一次操作次数</description>
 </property>   

```


## 查看Fsimage

```
hdfs oiv : apply the offline fsimage viewer to an fsimage

基本语法：
hdfs oiv -p 转换后的文件类型 -i 镜像文件 -o 转换后文件输出路径

lowkeysp-00@lowkeysp-01:/opt/module/hadoop-3.3.1/data/dfs/name/current$ hdfs oiv -p xml -i fsimage_0000000000000000342 -o ~/fsimage.xml
2021-07-14 15:44:26,738 INFO offlineImageViewer.FSImageHandler: Loading 5 strings
2021-07-14 15:44:26,800 INFO namenode.FSDirectory: GLOBAL serial map: bits=29 maxEntries=536870911
2021-07-14 15:44:26,800 INFO namenode.FSDirectory: USER serial map: bits=24 maxEntries=16777215
2021-07-14 15:44:26,800 INFO namenode.FSDirectory: GROUP serial map: bits=24 maxEntries=16777215
2021-07-14 15:44:26,800 INFO namenode.FSDirectory: XATTR serial map: bits=24 maxEntries=16777215

查看fsimage.xml，里面存储了文件目录的信息。


lowkeysp-00@lowkeysp-01:~$ cat fsimage.xml 
<?xml version="1.0"?>
<fsimage><version><layoutVersion>-66</layoutVersion><onDiskVersion>1</onDiskVersion><oivRevision>a3b9c37a397ad4188041dd80621bdeefc46885f2</oivRevision></version>
<NameSection><namespaceId>159680868</namespaceId><genstampV1>1000</genstampV1><genstampV2>1045</genstampV2><genstampV1Limit>0</genstampV1Limit><lastAllocatedBlockId>1073741869</lastAllocatedBlockId><txid>342</txid></NameSection>
<ErasureCodingSection>
<erasureCodingPolicy>
<policyId>1</policyId><policyName>RS-6-3-1024k</policyName><cellSize>1048576</cellSize><policyState>DISABLED</policyState><ecSchema>
<codecName>rs</codecName><dataUnits>6</dataUnits><parityUnits>3</parityUnits></ecSchema>
</erasureCodingPolicy>

<erasureCodingPolicy>
<policyId>2</policyId><policyName>RS-3-2-1024k</policyName><cellSize>1048576</cellSize><policyState>DISABLED</policyState><ecSchema>
<codecName>rs</codecName><dataUnits>3</dataUnits><parityUnits>2</parityUnits></ecSchema>
</erasureCodingPolicy>

<erasureCodingPolicy>
<policyId>3</policyId><policyName>RS-LEGACY-6-3-1024k</policyName><cellSize>1048576</cellSize><policyState>DISABLED</policyState><ecSchema>
<codecName>rs-legacy</codecName><dataUnits>6</dataUnits><parityUnits>3</parityUnits></ecSchema>
</erasureCodingPolicy>

<erasureCodingPolicy>
<policyId>4</policyId><policyName>XOR-2-1-1024k</policyName><cellSize>1048576</cellSize><policyState>DISABLED</policyState><ecSchema>
<codecName>xor</codecName><dataUnits>2</dataUnits><parityUnits>1</parityUnits></ecSchema>
</erasureCodingPolicy>

<erasureCodingPolicy>
<policyId>5</policyId><policyName>RS-10-4-1024k</policyName><cellSize>1048576</cellSize><policyState>DISABLED</policyState><ecSchema>
<codecName>rs</codecName><dataUnits>10</dataUnits><parityUnits>4</parityUnits></ecSchema>
</erasureCodingPolicy>

</ErasureCodingSection>

<INodeSection><lastInodeId>16468</lastInodeId><numInodes>56</numInodes><inode><id>16385</id><type>DIRECTORY</type><name></name><mtime>1625403554362</mtime><permission>lowkeysp-00:supergroup:0755</permission><nsquota>9223372036854775807</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16386</id><type>DIRECTORY</type><name>wcinput</name><mtime>1625326637268</mtime><permission>lowkeysp-00:supergroup:0755</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16387</id><type>FILE</type><name>aa.txt</name><replication>3</replication><mtime>1625326637243</mtime><atime>1625403562205</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741825</id><genstamp>1001</genstamp><numBytes>16</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16388</id><type>DIRECTORY</type><name>tmp</name><mtime>1625327231227</mtime><permission>lowkeysp-00:supergroup:0700</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16389</id><type>DIRECTORY</type><name>hadoop-yarn</name><mtime>1625327231227</mtime><permission>lowkeysp-00:supergroup:0700</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16390</id><type>DIRECTORY</type><name>staging</name><mtime>1625401809698</mtime><permission>lowkeysp-00:supergroup:0700</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16391</id><type>DIRECTORY</type><name>lowkeysp-00</name><mtime>1625327231227</mtime><permission>lowkeysp-00:supergroup:0700</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16392</id><type>DIRECTORY</type><name>.staging</name><mtime>1625403573223</mtime><permission>lowkeysp-00:supergroup:0700</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16393</id><type>DIRECTORY</type><name>job_1625325609725_0001</name><mtime>1625327231876</mtime><permission>lowkeysp-00:supergroup:0700</permission><xattrs><xattr><ns>SYSTEM</ns><name>hdfs.erasurecoding.policy</name><val>\0000;\0000;\0000;\000b;replication</val></xattr></xattrs><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16394</id><type>FILE</type><name>job.jar</name><replication>10</replication><mtime>1625327231734</mtime><atime>1625327231531</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741826</id><genstamp>1002</genstamp><numBytes>280989</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16395</id><type>FILE</type><name>job.split</name><replication>10</replication><mtime>1625327231847</mtime><atime>1625327231805</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741827</id><genstamp>1003</genstamp><numBytes>111</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16396</id><type>FILE</type><name>job.splitmetainfo</name><replication>3</replication><mtime>1625327231865</mtime><atime>1625327231849</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741828</id><genstamp>1004</genstamp><numBytes>25</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16397</id><type>FILE</type><name>job.xml</name><replication>3</replication><mtime>1625327232112</mtime><atime>1625327231876</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741829</id><genstamp>1005</genstamp><numBytes>232955</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16398</id><type>DIRECTORY</type><name>job_1625325609725_0002</name><mtime>1625327630899</mtime><permission>lowkeysp-00:supergroup:0700</permission><xattrs><xattr><ns>SYSTEM</ns><name>hdfs.erasurecoding.policy</name><val>\0000;\0000;\0000;\000b;replication</val></xattr></xattrs><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16399</id><type>FILE</type><name>job.jar</name><replication>10</replication><mtime>1625327630390</mtime><atime>1625327630268</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741830</id><genstamp>1006</genstamp><numBytes>280989</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16400</id><type>FILE</type><name>job.split</name><replication>10</replication><mtime>1625327630466</mtime><atime>1625327630440</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741831</id><genstamp>1007</genstamp><numBytes>111</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16401</id><type>FILE</type><name>job.splitmetainfo</name><replication>3</replication><mtime>1625327630891</mtime><atime>1625327630469</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741832</id><genstamp>1008</genstamp><numBytes>25</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16402</id><type>FILE</type><name>job.xml</name><replication>3</replication><mtime>1625327631080</mtime><atime>1625327630899</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741833</id><genstamp>1009</genstamp><numBytes>232955</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16403</id><type>DIRECTORY</type><name>job_1625325609725_0003</name><mtime>1625328819000</mtime><permission>lowkeysp-00:supergroup:0700</permission><xattrs><xattr><ns>SYSTEM</ns><name>hdfs.erasurecoding.policy</name><val>\0000;\0000;\0000;\000b;replication</val></xattr></xattrs><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16404</id><type>FILE</type><name>job.jar</name><replication>10</replication><mtime>1625328818879</mtime><atime>1625328818762</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741834</id><genstamp>1010</genstamp><numBytes>280989</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16405</id><type>FILE</type><name>job.split</name><replication>10</replication><mtime>1625328818971</mtime><atime>1625328818940</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741835</id><genstamp>1011</genstamp><numBytes>111</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16406</id><type>FILE</type><name>job.splitmetainfo</name><replication>3</replication><mtime>1625328818994</mtime><atime>1625328818973</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741836</id><genstamp>1012</genstamp><numBytes>25</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16407</id><type>FILE</type><name>job.xml</name><replication>3</replication><mtime>1625328819158</mtime><atime>1625328819000</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741837</id><genstamp>1013</genstamp><numBytes>232955</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16408</id><type>DIRECTORY</type><name>job_1625325609725_0004</name><mtime>1625329238333</mtime><permission>lowkeysp-00:supergroup:0700</permission><xattrs><xattr><ns>SYSTEM</ns><name>hdfs.erasurecoding.policy</name><val>\0000;\0000;\0000;\000b;replication</val></xattr></xattrs><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16409</id><type>FILE</type><name>job.jar</name><replication>10</replication><mtime>1625329238191</mtime><atime>1625329238067</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741838</id><genstamp>1014</genstamp><numBytes>280989</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16410</id><type>FILE</type><name>job.split</name><replication>10</replication><mtime>1625329238292</mtime><atime>1625329238260</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741839</id><genstamp>1015</genstamp><numBytes>111</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16411</id><type>FILE</type><name>job.splitmetainfo</name><replication>3</replication><mtime>1625329238322</mtime><atime>1625329238295</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741840</id><genstamp>1016</genstamp><numBytes>25</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16412</id><type>FILE</type><name>job.xml</name><replication>3</replication><mtime>1625329238551</mtime><atime>1625329238333</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741841</id><genstamp>1017</genstamp><numBytes>232946</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16413</id><type>DIRECTORY</type><name>job_1625399359435_0001</name><mtime>1625399632503</mtime><permission>lowkeysp-00:supergroup:0700</permission><xattrs><xattr><ns>SYSTEM</ns><name>hdfs.erasurecoding.policy</name><val>\0000;\0000;\0000;\000b;replication</val></xattr></xattrs><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16414</id><type>FILE</type><name>job.jar</name><replication>10</replication><mtime>1625399632357</mtime><atime>1625399631694</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741842</id><genstamp>1018</genstamp><numBytes>280989</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16415</id><type>FILE</type><name>job.split</name><replication>10</replication><mtime>1625399632464</mtime><atime>1625399632433</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741843</id><genstamp>1019</genstamp><numBytes>111</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16416</id><type>FILE</type><name>job.splitmetainfo</name><replication>3</replication><mtime>1625399632492</mtime><atime>1625399632467</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741844</id><genstamp>1020</genstamp><numBytes>25</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16417</id><type>FILE</type><name>job.xml</name><replication>3</replication><mtime>1625399632781</mtime><atime>1625399632503</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741845</id><genstamp>1021</genstamp><numBytes>232946</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16418</id><type>DIRECTORY</type><name>job_1625399359435_0002</name><mtime>1625400724265</mtime><permission>lowkeysp-00:supergroup:0700</permission><xattrs><xattr><ns>SYSTEM</ns><name>hdfs.erasurecoding.policy</name><val>\0000;\0000;\0000;\000b;replication</val></xattr></xattrs><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16419</id><type>FILE</type><name>job.jar</name><replication>10</replication><mtime>1625400724160</mtime><atime>1625400724058</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741846</id><genstamp>1022</genstamp><numBytes>280989</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16420</id><type>FILE</type><name>job.split</name><replication>10</replication><mtime>1625400724235</mtime><atime>1625400724209</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741847</id><genstamp>1023</genstamp><numBytes>111</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16421</id><type>FILE</type><name>job.splitmetainfo</name><replication>3</replication><mtime>1625400724259</mtime><atime>1625400724237</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741848</id><genstamp>1024</genstamp><numBytes>25</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16422</id><type>FILE</type><name>job.xml</name><replication>3</replication><mtime>1625400724414</mtime><atime>1625400724265</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741849</id><genstamp>1025</genstamp><numBytes>232949</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16428</id><type>DIRECTORY</type><name>history</name><mtime>1625403447744</mtime><permission>lowkeysp-00:supergroup:0755</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16429</id><type>DIRECTORY</type><name>done_intermediate</name><mtime>1625401809712</mtime><permission>lowkeysp-00:supergroup:1777</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16430</id><type>DIRECTORY</type><name>lowkeysp-00</name><mtime>1625403581266</mtime><permission>lowkeysp-00:supergroup:0770</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16431</id><type>DIRECTORY</type><name>wcoutput</name><mtime>1625401824315</mtime><permission>lowkeysp-00:supergroup:0755</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16438</id><type>FILE</type><name>part-r-00000</name><replication>3</replication><mtime>1625401824131</mtime><atime>1625401823963</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741856</id><genstamp>1032</genstamp><numBytes>17</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16440</id><type>FILE</type><name>_SUCCESS</name><replication>3</replication><mtime>1625401824318</mtime><atime>1625401824315</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><storagePolicyId>0</storagePolicyId></inode>
<inode><id>16443</id><type>FILE</type><name>job_1625401657661_0001-1625401805929-lowkeysp%2D00-word+count-1625401824326-1-1-SUCCEEDED-default-1625401812270.jhist</name><replication>3</replication><mtime>1625401824458</mtime><atime>1625401824421</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0770</permission><blocks><block><id>1073741858</id><genstamp>1034</genstamp><numBytes>22779</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16444</id><type>FILE</type><name>job_1625401657661_0001_conf.xml</name><replication>3</replication><mtime>1625401824534</mtime><atime>1625401824472</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0770</permission><blocks><block><id>1073741859</id><genstamp>1035</genstamp><numBytes>269287</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16445</id><type>DIRECTORY</type><name>done</name><mtime>1625403581238</mtime><permission>lowkeysp-00:supergroup:0770</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16452</id><type>DIRECTORY</type><name>output</name><mtime>1625403571872</mtime><permission>lowkeysp-00:supergroup:0755</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16458</id><type>FILE</type><name>part-r-00000</name><replication>3</replication><mtime>1625403571673</mtime><atime>1625403571442</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><blocks><block><id>1073741866</id><genstamp>1042</genstamp><numBytes>17</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16460</id><type>FILE</type><name>_SUCCESS</name><replication>3</replication><mtime>1625403571876</mtime><atime>1625403571872</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0644</permission><storagePolicyId>0</storagePolicyId></inode>
<inode><id>16463</id><type>FILE</type><name>job_1625403367618_0001-1625403546921-lowkeysp%2D00-word+count-1625403571935-1-1-SUCCEEDED-default-1625403554384.jhist</name><replication>3</replication><mtime>1625403572069</mtime><atime>1625403572023</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0770</permission><blocks><block><id>1073741868</id><genstamp>1044</genstamp><numBytes>22782</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16464</id><type>FILE</type><name>job_1625403367618_0001_conf.xml</name><replication>3</replication><mtime>1625403572142</mtime><atime>1625403572091</atime><preferredBlockSize>134217728</preferredBlockSize><permission>lowkeysp-00:supergroup:0770</permission><blocks><block><id>1073741869</id><genstamp>1045</genstamp><numBytes>269289</numBytes></block>
</blocks>
<storagePolicyId>0</storagePolicyId></inode>
<inode><id>16465</id><type>DIRECTORY</type><name>2021</name><mtime>1625403581238</mtime><permission>lowkeysp-00:supergroup:0770</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16466</id><type>DIRECTORY</type><name>07</name><mtime>1625403581238</mtime><permission>lowkeysp-00:supergroup:0770</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16467</id><type>DIRECTORY</type><name>04</name><mtime>1625403581238</mtime><permission>lowkeysp-00:supergroup:0770</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
<inode><id>16468</id><type>DIRECTORY</type><name>000000</name><mtime>1625403581266</mtime><permission>lowkeysp-00:supergroup:0770</permission><nsquota>-1</nsquota><dsquota>-1</dsquota></inode>
</INodeSection>
<INodeReferenceSection></INodeReferenceSection><SnapshotSection><snapshotCounter>0</snapshotCounter><numSnapshots>0</numSnapshots></SnapshotSection>
<INodeDirectorySection><directory><parent>16385</parent><child>16452</child><child>16388</child><child>16386</child><child>16431</child></directory>
<directory><parent>16386</parent><child>16387</child></directory>
<directory><parent>16388</parent><child>16389</child></directory>
<directory><parent>16389</parent><child>16390</child></directory>
<directory><parent>16390</parent><child>16428</child><child>16391</child></directory>
<directory><parent>16391</parent><child>16392</child></directory>
<directory><parent>16392</parent><child>16393</child><child>16398</child><child>16403</child><child>16408</child><child>16413</child><child>16418</child></directory>
<directory><parent>16393</parent><child>16394</child><child>16395</child><child>16396</child><child>16397</child></directory>
<directory><parent>16398</parent><child>16399</child><child>16400</child><child>16401</child><child>16402</child></directory>
<directory><parent>16403</parent><child>16404</child><child>16405</child><child>16406</child><child>16407</child></directory>
<directory><parent>16408</parent><child>16409</child><child>16410</child><child>16411</child><child>16412</child></directory>
<directory><parent>16413</parent><child>16414</child><child>16415</child><child>16416</child><child>16417</child></directory>
<directory><parent>16418</parent><child>16419</child><child>16420</child><child>16421</child><child>16422</child></directory>
<directory><parent>16428</parent><child>16445</child><child>16429</child></directory>
<directory><parent>16429</parent><child>16430</child></directory>
<directory><parent>16431</parent><child>16440</child><child>16438</child></directory>
<directory><parent>16445</parent><child>16465</child></directory>
<directory><parent>16452</parent><child>16460</child><child>16458</child></directory>
<directory><parent>16465</parent><child>16466</child></directory>
<directory><parent>16466</parent><child>16467</child></directory>
<directory><parent>16467</parent><child>16468</child></directory>
<directory><parent>16468</parent><child>16443</child><child>16444</child><child>16463</child><child>16464</child></directory>
</INodeDirectorySection>
<FileUnderConstructionSection></FileUnderConstructionSection>
<SecretManagerSection><currentId>0</currentId><tokenSequenceNumber>0</tokenSequenceNumber><numDelegationKeys>0</numDelegationKeys><numTokens>0</numTokens></SecretManagerSection><CacheManagerSection><nextDirectiveId>1</nextDirectiveId><numDirectives>0</numDirectives><numPools>0</numPools></CacheManagerSection>
</fsimage>

```

# 查看edits文件
```
hdfs oev -p 转换后文件类型 -i 编辑日志 -o 转换后文件输出路径

 hdfs oev -p xml -i edits_0000000000000000001-0000000000000000002 -o ~/edit_1.xml

```

# DataNode工作机制

一个数据块在DataNode上以文字形式存储在磁盘上，包括两个文件，一个是数据本身，`.meta`包含数据的长度，块数据的校验和，时间戳
```
lowkeysp-00@lowkeysp-01:/opt/module/hadoop-3.3.1/data/dfs/data/current/BP-725394105-127.0.1.1-1625319364440/current/finalized/subdir0/subdir0$ ll
总用量 4240
drwxrwxr-x 2 lowkeysp-00 lowkeysp-00   4096 7月   4 22:53 ./
drwxrwxr-x 3 lowkeysp-00 lowkeysp-00   4096 7月   3 23:37 ../
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00     16 7月   3 23:37 blk_1073741825
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00     11 7月   3 23:37 blk_1073741825_1001.meta
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00 280989 7月   3 23:47 blk_1073741826
-rw-rw-r-- 1 lowkeysp-00 lowkeysp-00   2203 7月   3 23:47 blk_1073741826_1002.meta

```

* 1）DataNode启动后向NameNode注册，通过后，周期性（6小时）向NameNode上报所有块信息
```
<property>
    <name>dfs.blockreport.intervalMsec</name>
    <value>21600000</value>
</property> 
```
DataNode再汇报前，会先自查自己本身的块信息，扫描自己节点块信息的时间间隔也是6小时
```
<property>
    <name>dfs.datanode.directoryscan.interval</name>
    <value>21600s</value>
</property> 
```
* 2)心跳是每3秒一次，告诉NameNode还活着，心跳返回的结果会包括NameNode给该DataNode的命令结果。如果超过10分钟+30s心跳没有收到某个DataNode的心跳，则认为该节点不可用。
 

 超时10分钟的计算：
 ```
 TimeOut = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval

 dfs.namenode.heartbeat.recheck-interval 默认是5分钟，dfs.heartbeat.interval默认是3秒
 ```