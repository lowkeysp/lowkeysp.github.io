---
title: SpringBoot-注解
date: 2021-11-22 17:00:54
tags:
---


# 组件注册到IOC容器中

生成配置类,
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Configuration //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {

    @Bean //给IOC容器中添加组件。方法名是组件id;返回类型是组件类型。返回的值是组件在容器中的实例，如：bean id为user01,
    public User user01(){
        return new User("张三",19);
    }

    @Bean
    public Pet tomcat(){
        return new Pet("tomcat");
    }
}
```

测试如下:
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {

        //返回IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        //返回IOC容器中注册的所有组件
        String[] beanDefinitionNames = run.getBeanDefinitionNames();

        for (String name : beanDefinitionNames) {
            System.out.println(name);
        }

    }
}
```
在输出中，会出现如下：
```
user01
tomcat
```

# 注册组件为单实例
测试代码如下：
```java

import com.example.boot.com.example.boot.User;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {

        //返回IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        //返回IOC容器中注册的所有组件
        //String[] beanDefinitionNames = run.getBeanDefinitionNames();

        User user01 = run.getBean("user01", User.class);
        User user02 = run.getBean("user01",User.class);
        System.out.println("uer01和user02是否一样: "+ (user01==user02));

    }
}
```

输出为：
```
uer01和user02是否一样: true
```
说明是单实例的，user01和user02实际上是同一个对象


# @Configuration的proxyBeanMethods属性
proxyBeanMethods属性有两个值，true和false，分别表示两种模式，full模式和lite模式
* Full模式：保证每个@Bean方法被调用多少次返回的组件都是单实例的
* Lite模式：每个@Bean方法被调用多少次返回的组件都是新创建的

测试如下：

`MyConfig.java`如下：
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class MyConfig {

    @Bean
    public User user01(){
        return new User("张三",19);
    }

    @Bean
    public Pet tomcat(){
        return new Pet("tomcat");
    }
}

```
`MainApplication.java`如下：
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {

        //返回IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        //返回IOC容器中注册的所有组件
        //String[] beanDefinitionNames = run.getBeanDefinitionNames();

        MyConfig config = run.getBean(MyConfig.class);
        System.out.println(config);

        User user01 = config.user01();
        User user02 = config.user01();

        System.out.println("uer01和user02是否一样: "+ (user01==user02));

    }
}
```
当`proxyBeanMethods = true`时，输出如下：
```
com.example.boot.MyConfig$$EnhancerBySpringCGLIB$$15ccf24f@6cd64ee8
uer01和user02是否一样: true
```
当`proxyBeanMethods = false`时，输出如下：
```
com.example.boot.MyConfig@75ae4a1f
uer01和user02是否一样: false
```

解释：当`proxyBeanMethods = true`时，如果调用`MyConfig的user01()方法`，则SpringBoot会检查IOC容器中是否有这个组件，如果有这个组件的话，则直接返回，没有才会创建，也就是full模式;当`proxyBeanMethods = false`时，则每次调用都是新创建的对象实例，也就是lite模式。


# @Import
`@Import({User.class})`

表示向容器中添加组件。在容器中组件的名字为全类名，比如上面添加了`User.class`，在IOC容器中的bean id 为：`com.example.boot.User`

另外，需要注意的是，Import注入时，需要类有无参构造器，否则会报错误。

# @Conditional注入

`@Conditional`叫作条件注入，满足指定的条件时，才会选择注入。

`@Conditional`有很多子类，比如:`@ConditionalOnBean`,` @ConditionalOnMissingBean`，`@ConditionalOnClass`等等，表示根据某些条件进行选择注入


举个例子：
```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = true)
public class MyConfig {
    
    @Bean
    public Pet tomcat(){
        return new Pet("tomcat");
    }

    @ConditionalOnBean(name = "tomcat")
    @Bean
    public User user01() {
        return new User("张三", 19);
    }
}
```
测试代码：
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {

        //返回IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);


        boolean user01 = run.containsBean("user01");
        boolean tomcat = run.containsBean("tomcat");

        System.out.println("user01是否注入：" + user01);
        System.out.println("tomcat是否注入: " + tomcat);

    }
}
```
则输出为：
```
user01是否注入：true
tomcat是否注入: true
```

如果把Pet的注入取消，
```java
    //@Bean
    public Pet tomcat(){
        return new Pet("tomcat");
    }
```
再次输出
```
user01是否注入：false
tomcat是否注入: false
```
由于tomcat没有注入到容器中，而user01的注入是根据tomcat来的，既然没有tomcat，则user01也不会注入

再进行测试时，发现了一个现象，就是如果`MyConfig.java`中，如果`user01`先于`pet`注入的话，则这个`ConditionalOnBean`不生效，应该是跟注入顺序有关，但是具体的原理还没查清楚。



