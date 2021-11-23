---
title: Spring-Nullable注解和函数式注解对象
date: 2021-11-17 10:55:45
tags:
---

# @Nullable注解

`@Nullable`注解可以使用在方法，属性，参数上，表示方法返回可以为空，属性值可以为空，参数值可以为空。

* 用在方法上，表示方法可以返回为空
```java

    @Nullable
    String getId();
```
* 用在方法的参数，表示方法的参数可以为空
```java
  public ClassPathXmlApplicationContext(String[] configLocations, @Nullable ApplicationContext parent) throws BeansException {
        this(configLocations, true, parent);
    }
```
* 用在属性上，表示属性值可以为空
```java
@Nullable
private String name;
```


# 函数式风格进行对象注册

在Spring中，不管是通过XML方式还是注解方式，都是将对象统一由IOC容器进行管理。

而对于自己创建的对象，比如`User user = new User()；`,则IOC容器是不会知道这个创建的对象的，如果需要将创建的对象注册到IOC容器中，可用下面方法进行注册
```java
        //1、创建GenericApplicationContext 对象
        GenericApplicationContext context = new GenericApplicationContext();
        //调用方法进行对象注册
        context.refresh();
        //第一个参数表示注册到IOC容器中的bean id
        context.registerBean("user1",User.class,()->new User());

        //获取在IOC容器中注册的对象
        User user1 = context.getBean("user1", User.class);
        System.out.println(user1);
```

