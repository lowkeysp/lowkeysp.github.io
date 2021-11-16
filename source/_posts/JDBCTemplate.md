---
title: JDBCTemplate
date: 2021-11-12 21:41:38
tags:
---


# 依赖和配置
```xml
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-tx -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.3.12</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework/spring-orm -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>5.3.12</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.12</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.26</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.8</version>
        </dependency>
```

在Spring配置文件中配置数据库连接池
```xml
<bean id="dataSource"
      class="com.alibaba.druid.pool.DruidDataSource"
      destroy-method="close">
    <property name="url" value="jdbc:mysql:///user_db"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>

</bean>
```

创建jdbcTemplates对象,注入DataSource。
```xml
    <!-- 创建jdbcTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 注入DataSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```

开启扫描
```xml
<context:component-scan base-package="com.example"></context:component-scan>
```

创建Service类和Dao类，Service里面注入Dao，Dao里面注入jdbcTemplate。

```java
import com.example.Dao.bookDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class bookService {
    //注入Dao
    @Autowired
    private bookDao bookDao;
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class bookDaoImpl implements bookDao{

    //注入jdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;
}
```

在`bookDaoImpl`里面操作数据库，
```java
//其中，sql是sql语句，args是设置sql语句中的参数
jdbcTemplate.update(String sql,Object...  args)

//用来查询
jdbcTemplate.queryForObject()

//批量添加
jdbcTemplate.batchUpdate()

```

在Service里面调用
```java
    @org.junit.Test
    public void Test(){

        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        UserService userService = context.getBean("userService", UserService.class);
        userService.transferMoney();
    }
```











