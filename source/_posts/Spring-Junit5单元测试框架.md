---
title: Spring-Junit5单元测试框架
date: 2021-11-17 16:34:59
tags:
---

Spring5框架整合了Junit5的单元测试，引入依赖
```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.13</version>

</dependency>

```

使用`SpringJunitConfig`,加载配置文件。
```java
@SpringJUnitConfig(locations = "classpath:bean.xml")
public class Test {

    @Autowired
    private UserService userService;

    @org.junit.Test
    public void Test(){
        userService.transferMoney();
    }

}
```


