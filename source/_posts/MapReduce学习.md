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

# 自定义序列化

如何实现自定义的序列化，可以先看看Hadoop已有的序列化类型

IntWritable

```java
public class IntWritable implements WritableComparable<IntWritable> 
```

是实现了一个`WritableComparable`的接口。这个接口里面有的内容是：
```java
public interface WritableComparable<T> extends Writable, Comparable<T> {
}
```

很尴尬，啥都没有，那么我们就再去看看继承的`Writable`和`Comparable`接口：
```java
public interface Writable {
    //Serialization method
    void write(DataOutput var1) throws IOException;
    //Deserialization method
    void readFields(DataInput var1) throws IOException;
}
```

这个接口需要实现两个方法，序列化方法和反序列化方法。

第二个接口：
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

那么，为什么要实现这个`Comparable`方法呢，这是因为MapReduce里的key是需要排序的（sorted）。因此，如果自定义的序列化要做key的话，必须实现这个接口，如果不做key的话，其实也可以忽略这个接口。

下面是一个自定义序列化`stuinfobean`的例子：

```java
public class StuInfoBean implements WritableComparable<StuInfoBean> {
    private Integer stuid;
    private String name;
    private Integer age;
    private Boolean sex;

    public StuInfoBean(){}

    public Integer getStuid() {
        return stuid;
    }

    public void setStuid(Integer stuid) {
        this.stuid = stuid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getSex() {
        return sex;
    }

    public void setSex(Boolean sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "StuInfoBean{" +
                "stuid=" + stuid +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", sex=" + sex +
                '}';
    }

    /**
     *Implement if you need to, skip if you don't need to
     */
    @Override
    public int compareTo(StuInfoBean o) {
        return this.stuid.compareTo(o.getStuid());
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeInt(stuid);
        dataOutput.writeUTF(name);
        dataOutput.writeInt(age);
        dataOutput.writeBoolean(sex);
    }

    /**
     *Note here that the order of the reverse sequence should be consistent with that of the serialization
     需要注意反序列化的顺序一定要和序列化的顺序一模一样！！
     */
    @Override
    public void readFields(DataInput dataInput) throws IOException {
        stuid = dataInput.readInt();
        name = dataInput.readUTF();
        age = dataInput.readInt();
        sex = dataInput.readBoolean();
    }
}

```



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

Mapper的实现通过`Job.setMapperClass(Class)`方法传给任务，Mapreduce框架会为每一个K，V对调用`map(WritableComparable, Writable, Context)`。还可以重写`cleanup(Context)`方法执行清理。

Mapper的输出KV对不需要跟输入的KV对类型一致，一个输入KV对可以映射成0到多个输出KV对。通过`context.write(WritableComparable, Writable)`来生成输出KV对，用来给Reduce做输入。

Mapper的同一个Key的所有value值，会被框架进行分组（group），然后传给Reduce，当然，可以通过`Job.setGroupingComparatorClass(Class)`指定一个`Comparator`来控制分组

Mapper的输出是排序好的，然后会进行分片（Partition），Mapper的分片数量实际上就是reduce任务的数量。可以通过实现自己的`Partitioner`来实现分片.(分片应该是为了防止内存溢出)。

用户还可以通过`Job.setCombinerClass(Class)`指定`combiner`(可选项)，进行一个局部的聚合，这样可以减少从Mapper到Reducer的数据传输

# Reducer

reduce的数量可以通过`Job.setNumReduceTasks(int)`设置

跟Mapper一样，通过`Job.setReducerClass(Class)`将reduce传给任务，然后会对每一个`<key, (list of values)>`调用`reduce(WritableComparable, Iterable<Writable>, Context)`,还可以重写`cleanup(Context)`方法。

Reducer主要由三个阶段：shuffle，sort和reduce

## shuffle
当Map任务产生output时，output会根据key进行排序，而且会将outputs传输到reducer所在的节点上。这整个过程为shuffle。通俗来讲，shuffle是map和reduce之间的数据处理过程。

当map任务开始产生output时，output首先会被写到一个内存缓存中，这个内存缓存默认是100MB。当然，这个内存缓存的值也可以通过`mapred-site.xml`文件中的`mapreduce.task.io.sort.mb parameter`进行更改。

当内存缓存达到一个阈值时，在内存缓存中的output会被溢出到磁盘中去。这个阈值默认是分配内存缓存大小的80%。当然，这个值也可以通过`mapreduce.map.sort.spill.percent`进行修改。当达到阈值时，就会在后台起一个线程，将output溢出到磁盘中去。

在output写到磁盘之前，会执行下面的动作：
* output会被分成多个partition（分区），分区的数量通常是reducer的数量。比如，如果有4个reducer，那么每一个map任务的output会被分成4个partition。一个partition里可以包含多个key,但是一个key的所关联的数据必须在一个partition里面。如果有10个mapper，每个mapper产生的output都会分成4个partition，然后一个partition会被传输到一个reducer中，partition默认是hashpartition。
* 在每一个partition中，数据也是根据key排序好的
* 如果定义了combiner，则会在写到磁盘前执行combiner。

因此，每当内存缓存达到阈值时，内存缓存中的数据就会生成一个新的溢出文件，然后上面的三个动作会被执行，然后被写到磁盘中去。之后，这些一个一个溢出到磁盘的文件会合并成一个文件，合并后的文件仍然遵守partition的界限以及每个partition中的排序

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gtgqffj7xjj318k0sctff.jpg)

当map的output被写到本地磁盘（Map任务运行的节点所在的）后，partition会被传送到reducer。每一个reducer都会接收到自己特定的partition。比如，如果有4个mapper和2个reducer，那么这些4个map的output会被分成2个partition，每个reducer一个。
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gtgqng4sh9j315f0jdqai.jpg)

一旦map任务结束，并且通知ApplicationMaster后，reduce任务则开始复制map传来的这些output。其实不用等到所有的map任务都结束。reducer并行的复制map的output。并行的数量默认是5.可以通过`mapreduce.reduce.shuffle.parallelcopies`来设置

在Reduce侧，数据也保存在内存缓存中。内存缓存的大小通过`mapreduce.reduce.shuffle.input.buffer.percent`设置。这个参数表示从最大的heap大小中可以分配到的百分比用于存储map的output。默认值是70%。

如果内存缓存存不了这么多数据，那么会溢出到磁盘中。阈值通过以下两个参数设置：
* 1）mapreduce.reduce.merge.inmem.threshold。如果在内存中合并的文件数量超过某一阈值时，则会溢出到磁盘中，默认值是1000
* 2）mapreduce.reduce.shuffle.merge.percent。表示超过内存缓存的大小的某一百分比，则会溢出到磁盘中。

一旦mapper产生的output被复制到reducer，而且合并成一个单独的排序好的文件后，这个就作为了reduce任务的输入

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gtgr1nyq68j31b10pkqbe.jpg)







## sort
框架会通过Key将相同的Key的输入进行分组,因为不同的mapper可能会生成同样的key。

shuffle和sort几乎是同时进行的。


## reduce
调用`reduce(WritableComparable, Iterable<Writable>, Context)`方法，输出通过`Context.write(WritableComparable, Writable)`写到文件系统中，而且Reducer的输出是不排序的。


# Partitioner
Partitioner是对key空间进行分片，准确的说，是对map的输出的key进行分片。通常使用Hash函数进行分片。`HashPartitioner`是默认的`Partitioner`

# Counter

Mapper and Reducer implementations can use the Counter to report statistics

# Combiner

Combiner又叫做Mini-Reducer,该组件的主要作用是：当我们执行mapreduce任务时，Mapper会产生大量的中间数据，这些中间数据需要传送到Reducer进行下一步处理，而传送则会带来很大的带宽压力，因此，Combiner的作用就是为了减轻这些带宽压力。Combiner是可选非必须的。在Mapper之后，在Reducer之前。

下图是没有使用Combiner的流程图。可以看到，输入被拆分成两个Mapper，生成了9对key-value，也就是中间数据，之后，这些中间数据被送到reducer中进行进一步处理。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gtzysewkp3j60po0o3tjv02.jpg)


下图是使用了Combiner的流程图。可以看到，使用了combiner之后，甚至需要将4个key-value传送给reducer。需要值得注意的是，combiner是在mapper所在的机器上运行的。
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gtzyuur0juj60og0o1dqb02.jpg)





# 切片Split

* 1）一个Job的Map阶段并行度由客户端在提交Job时的切片数决定
* 2）每一个Split切片分配一个MapTask并行实例处理
* 3）默认情况下，切片大小==BlockSize
* 4）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

切片指的是对输入的数据进行一个切片，是逻辑上的切片。针对每一个切片，会生成一个map任务。

切片的产生是由`InputFormat.class`类来实现的。这是一个抽象类
```java
public abstract class InputFormat<K, V> {
    public InputFormat() {
    }

    public abstract List<InputSplit> getSplits(JobContext var1) throws IOException, InterruptedException;

    public abstract RecordReader<K, V> createRecordReader(InputSplit var1, TaskAttemptContext var2) throws IOException, InterruptedException;
}
```
`FileInputFormat.class`类继承了这个`InputFormat.class`,`FileInputFormat.class`也是一个抽象类，
```java
public abstract class FileInputFormat<K, V> extends InputFormat<K, V> 
```
具体地实现，是由`TextFileInputFormat`，`NLineInputFormat`等具体类实现。

如果想知道分片的大小具体是如何计算的，可以查看`org.apache.hadoop.mapreduce.lib.input.FileInputFormat class`中的`computeSplitSize`方法，如下：
```java
protected long computeSplitSize(long blockSize, long minSize, long maxSize) {
  return Math.max(minSize, Math.min(maxSize, blockSize));
}
```
其中，
* minSize：为mapreduce.input.fileinputformat.split.minsize，可在配置文件中设置，表示：最小的可切片的大小，默认值是0
* maxSize：mapreduce.input.fileinputformat.split.maxsiz，可在配置文件中设置，表示：最大的可切片的大小，默认值是：Long.MAX_VALUE

因此，可以看出，默认的切片大小等于Block的块大小

分片之后，每一片还会进一步分成records(也就是key-value对)。

这些input splits和records都是逻辑的。他们并不会实际保存数据，真正的数据都在HDFS上的block里，他们只是对这些block块上的数据的引用。可以查看`InputSplit.class`
```java
public abstract class InputSplit {
  public abstract long getLength() throws IOException, InterruptedException;

  public abstract String[] getLocations() throws IOException, InterruptedException;

  @Evolving
  public SplitLocationInfo[] getLocationInfo() throws IOException {
    return null;
  }
}
```
可以看到，InputSplit类有得到input split长度的方法，还有数据存储的实际位置。

因此，可得input split只是对数据存储在HDFS块上的一种逻辑表示，而实际的数据是存储在HDFS的Block块上。那么，为什么MapReduce需要这两种概念，而并不直接处理HDFS块上的数据呢？

因为直接处理HDFS块上的数据会出现越界的情况，比如有一个record，一部分在Block1上，另一部分在Block2上，那么在Block1上执行的map任务将只能获取到在Block1上的record，而获取不到Block2上的record，所以逻辑的input split可以用来处理越界问题，通过使用Record在Block1上的开始位置以及偏移量，从而可以获取到整个record。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gtgaxcuv2xj30in07275e.jpg)


 

# OutputFormat

Reducer输出key-value对后，需要使用outputformat类，来把数据保存在数据库，文件等。

OutputFormat的核心是重写`RecordWriter`,`RecordWriter`的作用为：将Reducer输出的数据写到输入文件中。

下面介绍几种OutputFormat类

* TextOutputFormat：这也是Hadoop默认的outputformat。在Text文件里面每一行都是一个key-value对，而且key，value可以是任何类型，因为TextOutputFormat会调用toString()方法把它们转换成String。Key和Value之间是通过`\tab`进行区分，这个也可以通过`MapReduce.output.textoutputformat.separator`进行配置。
* SequenceFileInputFormat：将output写成序列化文件，通常被用在MapReduce任务之间，当作中间数据格式，可以快速的序列化以及反序列化。将前一个mapreduce任务产生的output序列化后，可以快速地反序列化，当作下一个mapreduce地输入。
* SequenceFileAsBinaryOutputFormat：跟SequenceFileInputFormat类似，只不过是将key，values以二进制的形式序列化。
* MapFileOutputFormat：将输入写成mapFiles。在MapFiles里面的key必须是排序的，因此 需要在reducer中保证输出的kay要排序。
* MultipleOutputs：将output写成文件，这些文件的名字来自输出的key，value值，也可以是任意的string。
* LazyOutputFormat：有时候，即使没有输出，outputformat也会产生一个空的文件，因此，可以使用LazyOutputFormat，仅当有内容时才会创建文件。LazyOutputFormat是一个wrapper。
* DBOutputFormat：将输出写入关系数据库和HBase的输出格式。

## 自定义OutputFormat

创建自己的OutputFormat类，继承FileOutputFormat类,需要重写`getRecordWriter`方法，需要返回`RecordWriter`类。
```java

public class WordCountOutputFormat<K,V> extends FileOutputFormat<K, V> {


    @Override
    public RecordWriter<K, V> getRecordWriter(TaskAttemptContext job) throws IOException, InterruptedException {
        
        //获得job的配置
        Configuration conf = job.getConfiguration();
        //返回
        return new WordCountLineRecordWriter<>()
    }
}
```


写自己的`RecordWriter`
```java

public class WordCountLineRecordWriter<K, V> extends RecordWriter<K, V> {

 
    public WordCountLineRecordWriter() {
        //构造器
    }
   

    public synchronized void write(K key, V value) throws IOException {
        //写入操作
    }
    public synchronized void close(TaskAttemptContext context) throws IOException {
        //关闭操作
    }
}
```

驱动
```java

job.setOutputFormatClass(WordCountOutputFormat.class);

FileInputFormat.addInputPath(job, input);
//注意，FileOutputFormat需要输出一个_SUCCESS文件，这个setOutputPath是用来设置输出_SUCCESS文件的位置
FileOutputFormat.setOutputPath(job, output);
```


# MR支持的压缩编码

|  压缩格式   | 是否Hadoop自带 | 算法 | 文件扩展名 | 是否可切片 | 换成压缩格式后，原来的程序是否需要修改
|  ----  | ----  |
| DEFLATE | 是 | DEFLATE | .deflate | 否 | 不需要|
| Gzip | 是 | Gzip | .gz | 否 | 不需要|
| bzip2 | 是 | bzip2 | .bzip2 | 是 | 不需要|
| LZO | 否，需要安装 | LZO | .lzo | 是 | 需要建索引，还需指定输入格式|
| Snappy | 是 | Snappy | .snappy | 否 | 不需要|

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gvdhj2zb46j61fs0mh17w02.jpg)



|压缩格式|对应的编码/解码器|
|----|----|
|DEFLATE| org.apache.hadoop.io.compress.DefalutCodec |
|Gzip| org.apache.hadoop.io.compress.GzipCodec |
|bzip2| org.apache.hadoop.io.compress.BZip2Codec |
|LZO| com.hadoop.compression.lzo.LaopCodec |
|Snappy| org.apache.hadoop.io.compress.SnappyCodec |

在Hadoop中启用压缩，做以下配置

| 参数| 默认值 | 阶段 | 建议 | 
| ---- | ---- | ---- | ---- |

输入阶段：

可在`core-site.xml`中的`io.compression.codecs`中配置，可通过`hadoop checknative`命令行查看支持的压缩格式

mapper输入阶段：

在`mapred-site.xml`中，`mapreduce.map.output.compress`设置是否开启，默认是false，`mapreduce.map.output.compress.codec`设置编码格式，默认是`org.apache.hadoop.io.compress.DefalutCodec`

reducer阶段：

在`mapred-site.xml`中，`mapreduce.output.fileoutputformat.compress`设置是否开启，默认是false,`mapreduce.output.fileoutputformat.compress.codec`设置编码格式，默认是`org.apache.hadoop.io.compress.DefalutCodec`


 也可在driver驱动中设置压缩格式

  
