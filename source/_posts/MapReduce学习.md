---
title: MapReduce学习
date: 2021-07-15 10:35:00
tags:
---

# 常用数据序列化类型



|  Java类型   | HadoopWritable类型 |
|  ----  | ----  |
| Boolean  | BooleanWritable |
| Byte  | ByteWritable |
| Int  | IntWritable |
| Float  | FloatWritable |
| Long  | LongWritable |
| Double  | DoubleWritable |
| String  | Text |
| Map  | MapWritable |
| Array  | ArrayWritable |
| Null  | NullWritable |


# MapReduce编程规范
用户编写程序分为三个部分：Mapper，Reducer，Driver

## Mapper阶段
* 用户自定义的Mapper要继承自己的父类
* Mapper的输入数据是KV对的形式（KV的类型可自定义）
* Mapper中的业务逻辑写在map()方法中
* Mapper的输出数据是KV对的形式（KV的类型可自定义）
* map()方法（MapTask进程）对每一个<K,V>调用一次

## Reducer阶段
* 用户自定义的Reducer要继承自己的父类
* Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
*  Reducer的业务逻辑写在reduce()方法中
* ReduceTask进程对每一组相同k的<K，V>组调用一次reduce()方法

## Driver阶段
相当于YARN集群的客户端，用于提交我们整个程序到YARN集群中，提交的是封装了MapReduce程序相关运行参数的job对象




