---
title: SpringBoot-HelloWorld
date: 2021-11-22 09:54:41
tags:
---

使用Idea创建一个maven工程

`POM.xml`的配置如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.0</version>
    </parent>

    <!-- Additional lines to be added here... -->

</project>
```


加入classpath依赖
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

创建SpringBoot的入口程序
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


//表示SpringBoot的主程序
@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

创建一个controller
```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/*
@RestController是@Controller和@ResponseBody的合体
*/
@RestController
public class controller {

    @RequestMapping("/hello")
    public String helloworldController(){
       return "Spring Boot 2 Hello World!";
    }
}
```

将上面的程序打包成可执行的jar包

在POM.xml文件中加入
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.6.0</version>
        </plugin>
    </plugins>
</build>
```
之后，点击`clean`和`package`，会在target里面生成一个`boot-01-helloworld-1.0-SNAPSHOT.jar`文件

在命令行输入：`java -jar boot-01-helloworld-1.0-SNAPSHOT.jar`，发现同样可以运行
```shell
D:\IdeaProjects\boot-01-helloworld\target>java -jar boot-01-helloworld-1.0-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.6.0)

2021-11-22 10:46:05.725  INFO 26032 --- [           main] com.example.boot.MainApplication         : Starting MainApplication v1.0-SNAPSHOT using Java 15 on SunPeng with PID 26032 (D:\IdeaProjects\boot-01-helloworld\target\boot-01-helloworld-1.0-SNAPSHOT.jar started by PC in D:\IdeaProjects\boot-01-helloworld\target)
2021-11-22 10:46:05.728  INFO 26032 --- [           main] com.example.boot.MainApplication         : No active profile set, falling back to default profiles: default
2021-11-22 10:46:07.066  INFO 26032 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-11-22 10:46:07.086  INFO 26032 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-11-22 10:46:07.087  INFO 26032 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.55]
2021-11-22 10:46:07.153  INFO 26032 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-11-22 10:46:07.153  INFO 26032 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1358 ms
2021-11-22 10:46:07.547  INFO 26032 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-11-22 10:46:07.560  INFO 26032 --- [           main] com.example.boot.MainApplication         : Started MainApplication in 2.328 seconds (JVM running for 2.716)
2021-11-22 10:46:29.505  INFO 26032 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-11-22 10:46:29.505  INFO 26032 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2021-11-22 10:46:29.508  INFO 26032 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms

```

还可以创建配置文件，用来简化配置,在`resource`目录下，创建`application.properties`。

比如如果要更改端口，则如下：
```
server.port=8888
```
具体的可配置的参数，可以参考官网
https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties


