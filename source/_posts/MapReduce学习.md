---
title: MapReduce学习
date: 2021-07-15 10:35:00
tags:
---

<meta name="referrer" content="no-referrer" />

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

# WordCount案例

Mapreduce框架是以`<key,value>`对为输入，`<key,value>`对为输出。

而且key和value必须实现Writable接口，key还要实现WritableComparable接口。

一个MapReduce任务的流程为
```
(input) <k1, v1> -> map -> <k2, v2> -> combine -> <k2, v2> -> reduce -> <k3, v3> (output)
```

## WordCount v1.0

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;


/*
输入：
$ bin/hadoop fs -ls /user/joe/wordcount/input/
/user/joe/wordcount/input/file01
/user/joe/wordcount/input/file02

$ bin/hadoop fs -cat /user/joe/wordcount/input/file01
Hello World Bye World

$ bin/hadoop fs -cat /user/joe/wordcount/input/file02
Hello Hadoop Goodbye Hadoop


*/





public class WordCount {

  /*<Text,IntWritable>Map的输出<K,V>类型
  */
  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    //Map方法，一次处理一行，
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    /*
      <Key Value>是Map的输入，且Value是Text类型，Context是用来Map和Reduce之间的交互桥梁
    
    */
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      //根据空格进行划分，同时将每个单词输出为<<word>,1>
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
      /*
      first map emits：
      < Hello, 1>
      < World, 1>
      < Bye, 1>
      < World, 1>

      The second map emits
      < Hello, 1>
      < Hadoop, 1>
      < Goodbye, 1>
      < Hadoop, 1>
      
      
      */

    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();


    /*
    values是一个集合，包含了同样的key的所有的数，比如<Hello，[1,1]>
    最后输出结果:
    < Bye, 1>
    < Goodbye, 1>
    < Hadoop, 2>
    < Hello, 2>
    < World, 2>
    */
    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    /*Combiner方法，实际上用的就是Reducer方法，
    输出为：
    The output of the first map:
    < Bye, 1>
    < Hello, 1>
    < World, 2>
    The output of the second map:
    < Goodbye, 1>
    < Hadoop, 2>
    < Hello, 1>
    */

    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}

```

# Mapper
Mapper的作用：将输入的K，V对 映射成 中间临时的K，V对，用于Reduce的输入

Maps可以看成是独立的任务，

# 序列化