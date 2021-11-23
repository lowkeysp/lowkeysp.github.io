---
title: SpringBoot-依赖管理特性
date: 2021-11-22 11:17:39
tags:
---

<meta name="referrer" content="no-referrer" />

在SpringBoot的`pom.xml`文件中，通常引入如下依赖，则会把整个依赖都引入进来
```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.0</version>
    </parent>
```
> 注：POM文件中的parent标签表示：POM文件也是有继承的，当有A，B，C三个项目，A是B和C的父级，如果B和C有很多jar都是共用的，则可以放在父项目上，则B，C继承A就可以，继承的用法就是在各自的POM文件中使用Parent。

查找`org.springframework.boot`的父项目，如下：
```xml
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.6.0</version>
  </parent>
```
发现还有父项目，再往上找，如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>2.6.0</version>
  <packaging>pom</packaging>
  <name>spring-boot-dependencies</name>
  <description>Spring Boot Dependencies</description>
  <url>https://spring.io/projects/spring-boot</url>
  <licenses>
    <license>
      <name>Apache License, Version 2.0</name>
      <url>https://www.apache.org/licenses/LICENSE-2.0</url>
    </license>
  </licenses>
  <developers>
    <developer>
      <name>Pivotal</name>
      <email>info@pivotal.io</email>
      <organization>Pivotal Software, Inc.</organization>
      <organizationUrl>https://www.spring.io</organizationUrl>
    </developer>
  </developers>
  <scm>
    <url>https://github.com/spring-projects/spring-boot</url>
  </scm>
  <properties>
    <activemq.version>5.16.3</activemq.version>
    <antlr2.version>2.7.7</antlr2.version>
    <appengine-sdk.version>1.9.92</appengine-sdk.version>
    <artemis.version>2.19.0</artemis.version>
    <aspectj.version>1.9.7</aspectj.version>

......(后续还有，太多了)
```
在这个文件里，指定了相关SpringBoot所需要的依赖以及版本号。

如果不想使用SpringBoot提供的依赖的版本号，想使用特定的版本号，可以如下实现:比如要更改mysql的版本，首先先在`spring-boot-dependencies`搜一下当前依赖的版本用的key，然后再自己的pom文件进行重写覆盖。
```xml
    <properties>
        <mysql.version>5.1.43</mysql.version>
    </properties>
```

在`POM.xml`文件中，还有一个依赖为：
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gwnxxgp171j30t20g9jve.jpg)

这种`spring-boot-starter-*`表示针对某种场景引入相关所有的依赖。比如`spring-boot-starter-aop`表示aop场景所需要的所有依赖，`spring-boot-starter-cache`表示Spring框架的cache所需要的依赖，`spring-boot-starter-web`表示Spring的web相关的所需要的依赖。具体可参考官方文档：https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using

也可以引用第三方的starter，第三方的命名官方建议不要使用`spring-boot`开头,可以使用`thirdpartyproject-spring-boot-starter`


