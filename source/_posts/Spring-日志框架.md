---
title: Spring-日志框架
date: 2021-11-17 10:37:45
tags:
---
Spring5框架已经移除了Log4jConfigListener,官方建议使用Log4j2

相关依赖引入:
```xml
<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.14.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.14.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.14.1</version>
    <scope>test</scope>
</dependency>


<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>

```

创建`log4j2.xml`配置文件，其中，`log4j2.xml`文件名称不能改变,关于`log4j2.xml`，网上有很多详细的介绍,本篇不再详细介绍
```xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 <!--日志级别以及优先级排序，OFF>FATAL>ERROR>WARN>INFO>DEBUG>TRACE>ALL -->
 2 <Configuration status="WARN">
 3   <Appenders>
 4     <Console name="Console" target="SYSTEM_OUT">
        <!-- 控制日志输出的格式-->
 5       <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
 6     </Console>
 7   </Appenders>
    <!--定义logger，只有定义了logger并引入的appender，appender才会生效-->
 8   <Loggers>
       <!-- root：用于指定项目的根日志，如果没有单独指定logger，则会使用root作为默认的日志输出-->
 9     <Root level="error">
10       <AppenderRef ref="Console"/>
11     </Root>
12   </Loggers>
13 </Configuration>
```

