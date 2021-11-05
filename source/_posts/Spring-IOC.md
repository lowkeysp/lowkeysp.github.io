---
title: Spring IOC
date: 2021-10-25 14:47:54
tags:
---
<meta name="referrer" content="no-referrer" />

Spring提供的容器又称为IoC容器，IoC全称Inversion of Control，直译为控制反转。

传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制，在IoC模式下，控制权发生了反转，即从应用程序转移到了IoC容器，所有组件不再由应用程序自己创建和配置，而是由IoC容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件。为了能让组件在IoC容器中被“装配”出来，需要某种“注入”机制

因此，IoC又称为依赖注入（DI：Dependency Injection），它解决了一个最主要的问题：将组件的创建+配置与组件的使用相分离，并且，由IoC容器负责管理组件的生命周期。

因为IoC容器要负责实例化所有的组件，因此，有必要告诉容器如何创建组件，以及各组件的依赖关系。一种最简单的配置是通过XML文件来实现

## 基于XML创建对象

创建一个xml配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- id是唯一标识，class表示类全路径，创建对象时，默认选择无参构造器 -->
    <bean id="book" class = "com.example.beanlearning.book">
    </bean>
</beans>
```
之后在java程序中调用
```java
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        book book = (book)context.getBean("book");
```

## 基于xml注入属性
上述只适用于无参构造器，不能带属性，如果创建对象时想带属性的话，用下面方法

### 使用set方法注入

类需要生成set方法，之后在xml中配置

```xml
    <bean id="book" class = "com.example.beanlearning.book">
        <property name="name" value="Springboot"></property>
        <property name="price" value="10"></property>

    </bean>

```

### 使用有参构造器注入

首先需要创建有参构造器，之后在xml中配置
```xml

    <bean id="book" class = "com.example.beanlearning.book">
        <constructor-arg name="name" value="hadoop"></constructor-arg>
        <constructor-arg name="price" value="200"></constructor-arg>
    </bean>
```
### p名称空间注入

可以简化xml注入属性,加上`xmlns:p="http://www.springframework.org/schema/p"`,这种方式需要类里面没有有参构造器，不然会报错
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="book" class = "com.example.beanlearning.book" p:name="NBA" p:price="100">
    </bean>
</beans>
```

### 注入特殊符号的属性和null属性
```xml
    <bean id="book" class = "com.example.beanlearning.book" >
        <!-- 设置null  -->
        <property name="price">
            <null/>
        </property>
        <!-- 设置特殊符号，可以使用 <![CDATA[ 特殊符号]]>  -->
        <property name="name">
            <value><![CDATA[<<南京>>]]></value>
        </property>
    </bean>
```

### 注入属性-外部bean

如果在一个类中需要注入一个类，跟注入属性一样，只不过将value换成了ref（引用）。可以使用set方法,在类中创建set方法，在xml中配置，这样，就实现了在user类中注入了book类，
```xml
    <bean id="book" class = "com.example.beanlearning.book" >
        <property name="name" value="Hadoop"></property>
        <property name="price" value="100"></property>
    </bean>

    <bean id="user" class = "com.example.beanlearning.User">
        <property name="book" ref="book"></property>
    </bean>
```
### 注入属性-内部Bean

上面是外部bean，user类中引用了book，在xml文件中是先定义了两个类，然后再用ref引用，还可以使用内部bean，如下：
```xml
    <bean id="user" class = "com.example.beanlearning.User">
        <property name="book">
            <bean id="book" class = "com.example.beanlearning.book" >
                <property name="name" value="Hadoop"></property>
                <property name="price" value="100"></property>
            </bean>
        </property>
```

### 注入集合属性

```xml
    <bean id="user" class = "com.example.beanlearning.User">

        <!--  注入数组类型 -->
        <property name="bookArray">
            <array>
                <value>Hadoop</value>
                <value>Spring</value>
            </array>
        </property>

        <!--  注入List类型 -->
        <property name="bookList">
            <list>
                <value>Hadoop</value>
                <value>Spring</value>
            </list>
        </property>

        <!--  注入map类型 -->
        <property name="bookMap">
            <map>
                <entry key="Hadoop" value="Hadoop"></entry>
                <entry key="Spring" value="Spring"></entry>
            </map>
        </property>

        <!--  注入set类型 -->
        <property name="sets">
            <set>
                <value>Hadoop</value>
                <value>Spring</value>
            </set>
        </property>
    </bean>
```
如果要在集合中加入的是对象的话，则可以将value换成ref,比如
```xml
        <property name="bookList">
            <list>
                <ref bean="book1"></ref>
                <ref bean="book2"></ref>
            </list>
        </property>
```
如果要在xml中直接创建list，而不是像上面一样在对象里面创建list的话，可用下面方法，
首先在xml头部加入：`xmlns:util="http://www.springframework.org/schema/util"`和`http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd`,之后，如下:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">


    <util:list id="bookList">
        <value>Hadoop</value>
        <value>Spring</value>
    </util:list>

    <bean id="user" class = "com.example.beanlearning.User">
        <property name="bookList" ref="bookList"> </property>
    </bean>
</beans>
```

### FactoryBean

FactoryBean主要用于xml文件中可以返回不同的Bean类型。举个例子：
```xml
<bean id="user" class = "com.example.beanlearning.User">
```
```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
User user = context.getBean("user",User.class);
```
表示使用user获取对象时，只能返回User对象，因为xml中的bean的class就规定了是User，但是可以通过继承FactoryBean，则可以返回非User对象
```java
public class User implements FactoryBean<T> {
    @Override
    public Object getObject() throws Exception {
        return null;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return FactoryBean.super.isSingleton();
    }
}
```
可以通过重写getObject()，从而返回其他对象，继承时，使用的是泛型。

### Bean的作用域

Bean的作用域常用的有Singleton和Prototype两种，其中，Singleton表示创建单实例，Prototype表示创建多实例，默认是Singleton

举个例子,创建两个user对象，user1和user2，输出，会发现地址都是一样的，
```xml
    <bean id="user" class = "com.example.beanlearning.User" scope="singleton">
    </bean>
```
```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");

User user1 = context.getBean("user",User.class);
User user2 = context.getBean("user",User.class);
System.out.println(user1);
System.out.println(user2);
```
输出：
```
com.example.beanlearning.User@64d2d351
com.example.beanlearning.User@64d2d351
```

如果设置成Prototype，
```xml
    <bean id="user" class = "com.example.beanlearning.User" scope="prototype">
    </bean>
```
输出为:
```
com.example.beanlearning.User@4461c7e3
com.example.beanlearning.User@351d0846
```
会发现输出的地址不一样，说明是创建了两个User对象。

* 如果设置scope为singleton时，加载spring配置文件时就会创建单实例对象
* 如果设置scope为prototype时，则是在调用getBean()方法的时候创建多实例对象

### Bean的生命周期
* 实例化 Instantiation
* 属性赋值 Populate
* 初始化 Initialization
* 销毁 Destruction

### Bean的自动装配
关于xml的自动装配一般有两种，byName和byType

byName表示根据属性名称注入，比如user中注入了book属性，在user类中定义的book属性名称跟xml中的book的bean id一致的话，则会自动装配
```xml
    <bean id="user" class = "com.example.beanlearning.User" autowire="byName">
    </bean>

    <bean id="book" class = "com.example.beanlearning.book"></bean>
```
byType表示根据属性类型注入，类型一样的则可以自动注入，但是如果有两个一样的类型的话，则会报错
```
    <bean id="user" class = "com.example.beanlearning.User" autowire="byType">
    </bean>

    <bean id="book" class = "com.example.beanlearning.book"></bean>
```
如下则会报错,因为book和book1都是book类型，使用byType无法知道该选择哪一个，只能使用byName。
```
    <bean id="user" class = "com.example.beanlearning.User" autowire="byType">
    </bean>

    <bean id="book" class = "com.example.beanlearning.book"></bean>
    <bean id="book1 class = "com.example.beanlearning.book"></bean>
```

### 引入外部属性文件

创建外部属性文件,比如test.properties
```
book.name="Hadoop"
book.price="100"
```
在xml中配置,首先需要添加命名空间，`xmlns:context="http://www.springframework.org/schema/context` 和 `http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 加载配置文件 -->
    <context:property-placeholder location="test.properties"/>

    <bean id="book" class = "com.example.beanlearning.book">
        <!-- 引用配置文件的数据  -->
        <property name="name" value="${book.name}"></property>
        <property name="price" value="${book.price}"></property>
    </bean>
</beans>
```










## 注解方式


有四种注解
* @Component
* @Service
* @Controller
* @Repository
### 注解方式创建对象

首先需要在xml文件中添加命名空间，`xmlns:context="http://www.springframework.org/schema/context`和`http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd`

之后，开启组件扫描，如果扫描多个包，则可以使用逗号隔开。
```xml
 <context:component-scan base-package="com.example.beanlearning"></context:component-scan>
```

之后在类名上添加Component，其中，`@Component(value = "book")`中的book相当于`<bean id = "book">`的bean中的id。value其实可以不写，不写的话就默认是类名，然后首字母小写。
```java
@Component(value = "book")
public class book {


}
```
之后就可以获取对象了
```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
book book = context.getBean("book",book.class);
```

### 组件扫描

 组件扫描可以进行过滤
 ```xml

    <!-- use-default-filters = false 表示不适用默认过滤器，下面过滤器定义了只扫描符合条件的-->
<context:component-scan base-package="com.example.beanlearning" use-default-filters="false">
        <context:include-filter type="annotation" 
                                expression="org.springframework.stereotype.Controller"/>

    </context:component-scan>


    <!-- 表示符合条件的都不扫描-->
<context:component-scan base-package="com.example.beanlearning" >
        <context:exclude-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>

    </context:component-scan>
 ```



 ### 使用注解注入属性

使用`@Autowired`进行自动装配，注入对象属性。会根据类型进行装配
```java
@Component
public class User {

    @Autowired
    private Book book;
}
```
使用`@Autowired`只能根据类型自动装配，会出现匹配多个的情况，则可以借助`@Qualifier`,根据value进行注入，注意：`@Qualifier`必须和`@Autowired`同时使用
```java

@Component(value = "book")
public class Book {

}

@Component
public class User {

    @Autowired
    @Qualifier(value = "book")
    private Book book;
}
```

`@Resource`可以根据类型注入，跟`@Autowired`一样，也可以根据名称注入，跟`@Qualifier`一样。

`@Value`用来注入普通类型的属性，比如String等，下面表示了name = “abc”
```java
@Value(value = "abc")
private String name;
```

### 完全注解
上面还需要使用xml文件配置，比如需要配置`<context:component-scan base-package="com.example.beanlearning"></context:component-scan>`

也可以不用使用xml配置文件，完全使用注解，实现如下：

创建一个配置class，写上注解,`@Configuration`,`@ComponentScan(basePackages = {"com.example.beanlearning"})`
```java
@Configuration
@ComponentScan(basePackages = {"com.example.beanlearning"})
public class SpringConfig {

}

//加载Config文件
ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
User user = context.getBean("user",User.class);
```



