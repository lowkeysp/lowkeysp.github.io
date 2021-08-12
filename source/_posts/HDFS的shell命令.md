---
title: HDFS的shell命令
date: 2021-07-05 10:09:42
tags:
---

<meta name="referrer" content="no-referrer" />

```
# 创建目录
hadoop fs -mkdir /d

# 从本地剪切到HDFS上，注意是剪切，上传之后，本地就没有了
hadoop fs -moveFromLocal 本地文件  HDFS目录

# 从本地复制到HDFS上
hadoop fs -copyFromLocal 本地文件  HDFS目录

# 跟copyFromLocal作用一样
hadoop fs -put 本地文件  HDFS目录

# 追加一个文件到已经存在的文件末尾
hadoop fs -appendToFile 本地文件  HDFS文件

# 从HDFS下载到本地
hadoop fs -copyToLocal HDFS文件  本地目录

# 跟copyToLocal作用一样
hadoop fs -get HDFS文件  本地目录

# 显示目录信息
hadoop fs -ls HDFS目录

# 显示文件内容
hadoop fs -cat HDFS文件

# 修改权限
hadoop fs -chgrp
          -chmod
          -chown

# 创建目录
hadoop fs -mkdir

# 复制
hadoop fs -cp 

# 移动
hadoop fs -mv

# 显示文件尾的内容
hadoop fs -tail

# 删除
hadoop fs -rm

#递归删除
hadoop fs -rm -r

#统计文件夹的大小信息
hadoop fs -du

# 设置HDFS中文件的副本数量
hadoop fs -setrep 副本数量 HDFS文件


```
