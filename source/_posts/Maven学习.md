---
title: Maven学习
date: 2021-07-22 15:18:50
tags:
---

<meta name="referrer" content="no-referrer" />

# Maven约定的目录结构

一个简单的约定目录结构如下：

```
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

|                    |   |
|  ----              | ----  |
| src/main/java	     |     Application/Library sources，项目的java源代码|
| src/main/resources |	Application/Library resources，项目的资源，比如说property文件，springmvc.xml (放配置文件)|
|src/main/filters    |	Resource filter files |
|src/main/webapp     |	Web application sources|
|src/test/java       |	Test sources，项目的测试类，比如说Junit代码          |
|src/test/resources  |	Test resources，测试用的资源|
|src/test/filters    |	Test resource filter files|
|src/it	             |Integration Tests (primarily for plugins)|
|src/assembly        |	Assembly descriptors|
|src/site	         |Site|
|LICENSE.txt         |	Project's license|
|NOTICE.txt	         |Notices and attributions required by libraries that the project depends on|
|README.txt	         |Project's readme|
|src/target          |打包输出目录      |
|src/target/classes      |编译输出目录 |
|src/target/test-classes    |测试编译输出目录 |


# POM.xml（Project Object Model）文件


## 坐标

groupId，artifactId和version标识了一个项目的三要素，表示：`<groupId>:<artifactId>:<version>`,比如，下面这个就是：`com.mycompany.app:my-app:1`，在互联网中是唯一的，可在https://mvnrepository.com/中搜索相应的项目

如果POM文件中的仓库没有指定的话，则会继承`Super POM`,在`Super POM`文件中，默认依赖都从https://repo.maven.apache.org/maven2下载，当然，下载会比较慢。

可以通过命令`mvn help:effective-pom`查看Super POM默认配置

```xml
<project>

  <modelVersion>4.0.0</modelVersion> #固定的POM.xml的版本，当前为4.0.0，不进行更改
  <!-- 公司或者组织的唯一标志，并且配置时生成的路径也是由此生成， 如com.mycompany.app，maven会将该项目打成的jar包放本地路径：/com/mycompany/app -->
  <groupId>com.mycompany.app</groupId>

  <!-- 项目的唯一ID，一个groupId下面可能多个项目，就是靠artifactId来区分的 -->
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```

## 依赖

通过坐标，可以添加依赖项
```xml
  <dependencies>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-artifact</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-core</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
  </dependencies>
```

# 生命周期 Build Lifecycle

Maven的核心概念就是构建生命周期（build lifecycle）。这意味着构建和分发一个项目的过程是明确定义好的。

有三个内置的（built-in）构建生命周期（build lifecycles）：default，clean和site。其中，default生命周期处理项目的部署；clean生命周期处理项目的清理；site生命周期处理项目站点文档的生成。

这三个生命周期由一系列不同的构建阶段（build phases）组成

## Default Lifecycle

Default生命周期有以下几个阶段（phase）（不全，后面附上全部的阶段）：
* validate -- 验证项目是否正确且所有必要的信息是可用的
* compile -- 项目源代码的编译
* test -- 使用测试框架（如Junit）对编译好的代码进行测试
* package -- 将编译好的文件进行打包
* verify -- 对集成测试的结果进行检查，保证质量达标
* install -- 将打包好的保存到本地仓库
* deploy -- 部署到远程仓库中

这些lifecycle phases都是顺序执行。而且比如执行package，package之前的阶段也会被执行





|   Phase                 |  Description |
|  ----              | ----  |
|validate	|validate the project is correct and all necessary information is available.|
| initialize	|initialize build state, e.g. set properties or create directories.|
| generate-sources	|generate any source code for inclusion in compilation.|
|process-sources|	process the source code, for example to filter any values.|
|generate-resources |	generate resources for inclusion in the package.|
|process-resources	|copy and process the resources into the destination directory, ready for packaging.|
|compile	|compile the source code of the project.|
|process-classes|	post-process the generated files from compilation, for example to do bytecode enhancement on Java classes.|
|generate-test-sources	|generate any test source code for inclusion in compilation.|
|process-test-sources	|process the test source code, for example to filter any values.
|generate-test-resources	|create resources for testing.|
|process-test-resources	|copy and process the resources into the test destination directory.|
|test-compile	|compile the test source code into the test destination directory|
|process-test-classes|	post-process the generated files from test compilation, for example to do bytecode enhancement on Java classes.|
|test	|run tests using a suitable unit testing framework. These tests should not require the code be packaged or deployed.|
|prepare-package|	perform any operations necessary to prepare a package before the actual packaging. This often results in an unpacked, processed version of the package.|
|package|	take the compiled code and package it in its distributable format, such as a JAR.|
|pre-integration-test	|perform actions required before integration tests are executed. This may involve things such as setting up the required environment.|
|integration-test	|process and deploy the package if necessary into an environment where integration tests can be run.|
|post-integration-test|	perform actions required after integration tests have been executed. This may including cleaning up the environment.|
|verify|	run any checks to verify the package is valid and meets quality criteria.|
|install|	install the package into the local repository, for use as a dependency in other projects locally.|
|deploy	|done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.|



## Clean Lifecycle

| Phase |	Description|
|---|---|
|pre-clean	| execute processes needed prior to the actual project cleaning|
|   clean	  | remove all files generated by the previous build| 
|  post-clean |	execute processes needed to finalize the project cleaning| 

## Site Lifecycle


|  Phase	| Description|
|---|---|
| pre-site	| execute processes needed prior to the actual project site generation|
| site |	generate the project's site documentation|
|  post-site	| execute processes needed to finalize the site generation, and to prepare for site deployment | 
| site-deploy  | 	deploy the generated site documentation to the specified web server|

## Plugin Goal

A Build Phase is Made Up of Plugin Goals

一个build phase包含0个或者1个或者多个Plugin Goal。Plugin Goal可以表示一个特定的任务，真正执行用来构建和管理项目的。一个Plugin Goal也可以不绑定到build phase上，在生命周期之外直接调用。

举个例子：clean 和 package都是build phase，而dependency:copy-dependencies是一个goal（of a plugin）
```
mvn clean dependency:copy-dependencies package
```
首先，clean先被执行（意味着clean生命周期的clean阶段以及之前的阶段都被执行），接着执行`dependency:copy-dependencies`goal,最后执行package（意味着default生命周期的package以及之前的阶段都被执行）。

如果一个goal绑定到一个或者多个阶段，则在这些阶段里面都会调用这个goal。

当然，一个阶段可以包括0个或者多个goal，如果一个阶段里面没有任何goal的话，则这个阶段将不会被执行。（注意：如果多个goal绑定到一个阶段的话，则goal执行的顺序就是在POM文件中生命的顺序）



# Maven命令

* mvn compile

编译main/java目录下的所有Java文件为class文件，且class文件存放在target/classes目录下；将main/resources下的配置文件，拷贝到target/classes目录下

* mvn test-compile

编译test/java目录下的所有Java文件为class文件，且class文件存放在target/test-classes目录下;将test/resources下的配置文件，拷贝到target/test-classes目录下

* mvn test 

进行test的

* mvn package

打包，生成到target/目录下，只打包main下的目录，并不打包test下的目录

# 修改本地仓库的地址

修改`D:\tools\apache-maven-3.8.1\conf`下的`settings.xml`文件
注意，新的路径一定不能包含中文

可以看到，默认的路径是在用户home下的.m2的repository下。
```
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
```

# Maven在IDEA的配置

IDEA-->Settings-->Build,Execution,Deployment-->Build Tools-->Maven

* Maven home path: Maven的安装目录，本次为：D:\tools\apache-maven-3.8.1
* User settings file：Maven安装目录conf/settings.xml文件,本次为：D:\tools\apache-maven-3.8.1\conf\settings.xml
* Local Repository：本地仓库的目录位置，本次为：D:\tools\maven_repository\repository

在Runner里面，设置JDK，如果已经设置则不用管，在VM option中，设置`-DarchetypeCatalog=internal`,设置这一项，则创建Maven工程会快一些（默认会联网下载模板文件，该设置则不会下载）



# Maven过滤resource files

有时候，resource文件的一些参数需要在项目构建时才能确认，并无法提前确认，这时候，可以使用语法`${<property name>}`,这个`property name`可以是在POM.XML文件中定义好的，也可以是在setting.xml文件中定义好的,还可以是一个外部的配置文件定义好的，还可以是系统文件。

## 配置文件引用POM.XML中的参数

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
 
  <--! 这里设置过滤器为true，因为默认是false -->
  <build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
</project>
```

假设在/src/resources目录下有个配置文件`application.properties`,内容如下：

```
# application.properties
application.name=${project.name}
application.version=${project.version}

```

那么，在编译之后，或使用`mvn process-resources`,在`target/classes`目录下会生成一个文件`application.properties`,内容如下：
```
# application.properties
application.name=Maven Quick Start Archetype
application.version=1.0-SNAPSHOT
```

通常，`${project.name}`表示项目的名字，`${project.version}`表示项目的版本，`${project.build.finalName}`表示项目打包后的名称的final name。


## 配置文件引用setting.xml中的参数

同样，调用settings.xml中的参数也是同样的方法，需要以`settings`开头，比如`${settings.localRepository}`表示用户本地仓库的路径。

## 配置文件引用外部配置文件参数
调用外部的配置文件的话，需要做的就是在POM.xml中引用外部配置文件。比如，在`src/main/filters/filter.properties`创建一个外部配置文件

```
# filter.properties
my.filter.value=hello!
```

在POM.xml文件中,引用这个新的文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <build>
    <filters>
      <filter>src/main/filters/filter.properties</filter>
    </filters>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
</project>

```

然后，在`application.properties`中，内容如下：
```xml
# application.properties
application.name=${project.name}
application.version=${project.version}
message=${my.filter.value}
```

或者，在POM.XML的`properties`区中配置，然后引用，具体如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
 
  <build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
 
  <properties>
    <my.filter.value>hello</my.filter.value>
  </properties>
</project>

```

## 配置文件引用系统配置参数

系统配置参数比如：`java.version`,`user.home`，或者使用标准的java带参数（-D）的命令行。
```xml
# application.properties
java.version=${java.version}
command.line.prop=${command.line.prop}
```

命令为：
```
mvn process-resources "-Dcommand.line.prop=hello again"
```


