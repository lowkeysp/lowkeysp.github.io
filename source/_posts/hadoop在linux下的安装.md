---
title: hadoop在linux下的安装
date: 2020-11-27 09:16:40
tags: 
---

<meta name="referrer" content="no-referrer" />

首先，需要确认Linux系统是否装有JDK，如果没有的话，得先安装JDK

# Linux环境下安装JDK

* 从官网上下载Oracle JDK，此次下载的是 jdk-15.0.1_linux-x64_bin.tar.gz 

*  解压：sudo tar -zxvf jdk-15.0.1_linux-aarch64_bin\ 1.tar.gz -C /opt/modules/
> /opt 目录简单介绍：Optional application software packages，/opt目录用来安装附加软件包，是用户级的程序目录，可以理解为D:/Software。安装到/opt目录下的程序，它所有的数据、库文件等等都是放在同个目录下面。当你不需要时，直接rm -rf掉即可。

* 添加环境变量(此处需要使用root权限，因为/etc/profile 文件只能root文件写入),在/etc/profile文件中追加写入
```
export JAVA_HOME="/opt/modules/jdk-15.0.1"
export PATH=$JAVA_HOME/bin:$PATH
```
退出文件后，命令 `source /etc/profile` 使命令生效。

查看是否安装成功，用 
```
java -version
```


之后，就是Hadoop的安装了
# Hadoop安装

下载安装包,此处使用的是清华的镜像

https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.10.1/

## 本地模式

1、 在/opt/modules/下创建一个hadoopstandalone目录

```
sudo mkdir / opt/modules/hadoopstandalone
```

2、解压

```
sudo tar -zxvf hadoop-2.10.1.tar.gz -C /opt/modules/hadoopstandalone/
```


3、 运行mapreduce程序，验证。这是使用hadoop自带的wordcount例子来在本地模式下测试跑mapreduce。自己在/opt/data/下生成一个文件，wc.input，内容如下：

```
#cat /opt/data/wc.input
hadoop
mapreduce
hivehbase
spark
stormsqoop
hadoop
hivespark
hadoop
```

运行hadoop自带的mapreduce Demo

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar wordcount /opt/data/wc.input output2
```

去output2/目录下查看，
```
# ls
part-r-00000  _SUCCESS
```
其中,_SUCCESS文件说明JOB运行成功，part-r-00000是输出结果文件

查看part-r-00000
```
#cat part-r-00000 
hadoop	3
hivehbase	1
hivespark	1
mapreduce	1
spark	1
stormsqoop	1
```