---
title: Spring-事务
date: 2021-11-15 14:33:25
tags:
---

事务是操作库的基本单元，逻辑上一组操作，要么都成功，如果有一个失败，则所有操作都失败

* 原子性
* 一致性
* 隔离性
* 持久性

Spring的声明式事务管理，底层使用的是AOP原理



# Spring事务的配置

## 注解方式

* 第一步，创建事务管理器,注入datasource
```xml

    <bean id="dataSource"
          class="com.alibaba.druid.pool.DruidDataSource"
        destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3306/springlearning"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>

    </bean>


    <!-- 创建事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```
* 开启事务注解
```xml

<!-- 引入命名空间-->
    xmlns:tx="http://www.springframework.org/schema/tx"
    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd



    <!-- 开启事务的注解-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```

* 在service上加注解，`@Transactional`,`@Transactional`既可以添加到类上面，表示类的所有方法都添加了事务；也可以添加到方法上，表示只是给这个方法添加了事务
比如：
```java
    @Transactional
    public void transferMoney(){
        userDao.deleteMoney(100);
        userDao.addMoney(100);

    }
```

## @Transactional详解

### propagation-事务传播行为

```java
@Transactional(propagation = Propagation.REQUIRED)
@Transactional(propagation = Propagation.REQUIRES_NEW)
@Transactional(propagation = Propagation.SUPPORTS)
@Transactional(propagation = Propagation.NOT_SUPPORTED)
@Transactional(propagation = Propagation.MANDATORY)
@Transactional(propagation = Propagation.NEVER)
@Transactional(propagation = Propagation.NESTED)
```

* 1、PROPAGATION_REQUIRED(propagation_required)—支持当前事务，如果当前没有事务，就新建一个事务；方法A调用方法B，如果方法A的已经有开启事务了，那么方法B就不会再新开启事务，而是直接使用方法A的事务，不管双方哪一个执行失败，都会一起回滚。
* 2、PROPAGATION_REQUIRES_NEW(propagation_requires_new)—新建事务，如果存在当前事务，把当前事务挂起；方法A调用方法B，就算方法A已经开启了事务，那么方法B也依旧会开启一个新的事务，执行期间将方法A的事务挂起，直到执行完毕之后才开始继续执行方法A，如果方法A执行失败回滚，不会影响到方法B，方法B该提交还是一样提交。
* 3、PROPAGATION_SUPPORTS(propagation_supports)—支持当前事务，如果没有当前事务，就以非事务方式执行。此方式比较特殊，是一种随波逐流的方式，如果有它就用，如果没有它就不用。
* 4、PROPAGATION_NOT_SUPPORTED(propagation_not_supproted)—以非事务方式执行操作，如果当前存在事务，就把当前事务挂起；不支持事务，如果方法A已经开启了事务，调用方法B时，会将方法A的事务挂起之后开始执行方法B，执行完毕之后继续执行方法A。
* 5、PROPAGATION_MANDATORY(propagation_mandatory)—支持当前事务，如果当前没有事务，就抛出异常；强制在一个事务中执行，如果方法A开启了事务，方法B也试图开启一个事务，此时就会直接抛出异常。
* 6、PROPAGATION_NEVER(propagation_never)—以非事务方式执行，如果当前存在事务，则抛出异常；
不支持事务，如果强制开启事务，或者方法A已经开启了事务，再来调用方法B，就会直接抛出异常。
* 7、PROPAGATION_NESTED(propagation_nested)—新建事务，依赖于上级事务，如果失败则一同回滚；
这个事务和requires_new很相似，但是不同点在于事务提交的时候，它会让方法B依赖于方法A的事务，B就算开启了事务，也不会主动提交，而是等待A一起提交，此时如果A报错回滚了，那么B也会一起回滚。

### isolation-隔离

事务会存在的三个问题：

* 脏读
* 不可重复读
* 幻读



```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
@Transactional(isolation = Isolation.READ_COMMITTED)
@Transactional(isolation = Isolation.SERIALIZABLE)
```

事务隔离解决上述三个问题：
||脏读|不可重复读|幻读|
|--|--|--|--|
|Read UnCommitted(读未提交)|有|有|有|
|Read Committed(读提交)|无|有|有|
|Repeatable Read(重复读)|无|无|有|
|Serializable(序列化)|无|无|无|


### timeout

```java
@Transactional(timeout = -1)
```
事务需要在一定时间内提交，如果不提交则进行回滚，默认是-1，表示不超时


### readOnly
```java
@Transactional(readOnly = true)
```
表示是否只读，默认是false，表示可以查询，也可以删除添加修改

### rollbackFor
```java
@Transactional(rollbackFor = )
```
设置出现哪些异常进行事务回滚

### noRollbackFor
```java
@Transactional(noRollbackFor = )
```
设置出现哪些异常不进行事务回滚


## 基于XML方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">


    <bean id="dataSource"
          class="com.alibaba.druid.pool.DruidDataSource"
        destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3306/springlearning"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>

    </bean>

    <!-- 创建jdbcTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 注入DataSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    


    <!-- 创建事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 配置通知-->
    <tx:advice id="txadvice">
        <!-- 配置事务的参数-->
        <tx:attributes>
            <tx:method name="transferMoney" propagation="REQUIRES_NEW"/>
        </tx:attributes>
    </tx:advice>


    <!-- 配置切入点和切面-->
    <aop:config>
        <!-- 配置切入点-->
        <aop:pointcut id="pt" expression="execution(* com.example.Service.UserService.*(..)) "/>
        <!-- 配置切面,把txadvice通知到pt切入点上-->

        <aop:advisor advice-ref="txadvice" pointcut-ref="pt"></aop:advisor>
    </aop:config>
</beans>

```

## 基于完全注解



