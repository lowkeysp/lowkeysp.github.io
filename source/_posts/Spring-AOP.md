---
title: Spring-AOP
date: 2021-10-28 15:33:53
tags:
---

AOP，Aspect Oriented Programming（面向切面编程），利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

# 动态代理
`java.lang.reflect.Proxy`中的静态方法`newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`,表示返回一个代理实例，Returns a proxy instance for the specified interfaces that dispatches method invocations to the specified invocation handler.

下面举了一个例子，创建了一个Person接口，这个接口有两个方法，
```java
public interface Person {

    public int add(int a,int b);
    public int divide(int a ,int b);
}
```
Boy类继承了这个Person接口
```java
public class Boy implements Person{

    @Override
    public int add(int a,int b) {
        System.out.println("方法中执行");
        int c = a+b;
        return c;
    }
    @Override
    public int divide(int a,int b) {
        System.out.println("方法中执行");
        int c = a / b;

        return c;
    }
}
```
如果需要增强Boy中的方法，借助`java.lang.reflect.Proxy`和`java.lang.reflect.InvocationHandler`。

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class JDKProxy {

    public static void main(String[] args) {

        Class[] interfaces = {Person.class};

        //创建了一个代理实例,
        //ClassLoader loader 表示调用方法所在类的加载器
        //Class<?>[] interfaces 表示要加强的方法所在的类所继承的接口，可以是多接口
        //InvocationHandler h 具体实现要增强的方法
        Person p = (Person) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces,new handler(new Boy()));
        int  a = p.divide(1,2);
        System.out.println(a);
    }
}


class handler implements InvocationHandler {

    private Object obj;

    public handler(Object obj) {
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("方法"+ method.getName()+"之前执行。。。");
        Object res = method.invoke(obj,args);

        System.out.println("方法"+ method.getName()+"之后执行。。。");
        return res;
    }
}
```

# AOP术语
* 连接点

类里面哪些方法可以被增强，这些方法称为连接点

* 切入点

实际真正被增强的方法，这些方法称为切入点

* 通知（增强）

实际增强的逻辑部分称为通知（增强）。其中，通知有前置通知，后置通知，环绕通知，异常通知，最终通知

* 切面

把通知应用到切入点的过程，叫作切面，是一个动作。

# AOP操作
Spring框架通常都是基于AspectJ实现AOP操作。AspectJ并不是Spring的组成部分，是一个独立的AOP框架，一般把Spring和AspectJ一起使用进行AOP操作。

基于AspectJ实现AOP操作可以基于XML配置文件实现，也可以通过注解方式实现

切入点表达式：知道对哪个类里面的哪个方法进行增强

* 语法结构：

`execution([权限修饰符][返回类型][类全路径][方法名称]([参数列表]))`


## 基于AspectJ注解的AOP操作
假设创建一个类Boy，Boy类有add()方法，要增强这个add()方法。如何通过AspectJ实现？

* 创建Boy类，以及add()方法,同时增加`@Component`.
```java
import org.springframework.stereotype.Component;

@Component
public class Boy {
    public void add() {
        System.out.println("add()方法执行......");
    }
}
```
* 创建增强的类，下面涉及了五种通知（增强）类型

需要在增强的类上添加`@Componet`和`@Aspect`,`@Aspect`表示要生成代理对象

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class BoyProxy {

    //前置通知，value值后面是execution语法结构，表示切入点表达式，表示要增强的方法
    @Before(value = "execution(* com.example.beanlearning.Boy.add(..))")
    public void before(){
        System.out.println("前置通知。。。。" );
    }

    //后置通知
    @After(value = "execution(* com.example.beanlearning.Boy.add(..))")
    public void after(){
        System.out.println("后置通知。。。。" );
    }

    //环绕通知，环绕通知需要传入参数ProceedingJoinPoint，调用原来的add()方法
    @Around(value = "execution(* com.example.beanlearning.Boy.add(..))")
    public void round(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前。。。。" );
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后。。。。" );
    }

    //最后通知
    @AfterReturning(value = "execution(* com.example.beanlearning.Boy.add(..))")
    public void afterReturning(){
        System.out.println("最后通知。。。。" );
    }

    //异常通知
    @AfterThrowing(value = "execution(* com.example.beanlearning.Boy.add(..))")
    public void afterThrowing(){
        System.out.println("异常通知。。。。" );
    }
}
```

* 修改bean.xml创建(如果有的话，没有需要创建)


添加`xmlns:context`和`xmlns:aop`,`xsi:schemaLocation`,
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.example.beanlearning"></context:component-scan>
    <!-- 开启Aspect，生成代理对象-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

```java
    ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
    Boy boy = context.getBean("boy", Boy.class);
    boy.add();
```

输出为：
```
环绕之前。。。。
前置通知。。。。
add()方法执行......
最后通知。。。。
后置通知。。。。
环绕之后。。。。
```

当然，如果要完全注解的话，上面的xml配置还可以改成
```java
@Configuration
@ComponentScan(basePackages={"com.example.beanlearning"})
@EnableAspectJAutoProxy(proxyTargetClass=True)
public class Config{

}

```


上面为一个完整的AOP操作，但是存在一些可以改进的地方，比如在通知时，切入点表达式`execution(* com.example.beanlearning.Boy.add(..))`都是一样的，需要重复写好几遍，可以提取出来，具体做法，如下:

通过使用`@Pointcut`,从而只需要更改一处，就能实现所有更改。
```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class BoyProxy {

    @Pointcut(value = "execution(* com.example.beanlearning.Boy.add(..))")
    public void pointcut(){

    }

    @Before(value = "pointcut()")
    public void before(){
        System.out.println("前置通知。。。。" );

    }
    @After(value = "pointcut()")
    public void after(){
        System.out.println("后置通知。。。。" );
    }

    @Around(value = "pointcut()")
    public void round(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前。。。。" );
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后。。。。" );
    }

    @AfterReturning(value = "pointcut()")
    public void afterReturning(){
        System.out.println("最后通知。。。。" );
    }

    @AfterThrowing(value = "pointcut()")
    public void afterThrowing(){
        System.out.println("异常通知。。。。" );
    }
}
```


如果有多个增强类对同一个方法进行通知（增强），如何确定优先级？

在增强类上面添加注解`@Order(数字)`,数字越小，优先级越高，比如`@Order(1)`
```java
@Component
@Aspect
@Order(1)
public class BoyProxy{

}
```




## 基于配置文件进行AOP操作

创建Boy和BoyProxy类跟之前一样，只不过不需要加任何注解,在spring配置文件中，配置如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 创建对象 -->
    <bean id="boy" class="com.example.beanlearning.Boy"></bean>
    <bean id="boyProxy" class="com.example.beanlearning.BoyProxy"></bean>

    <!-- 配置AOP-->
    <aop:config>
        
        <!-- 切入点配置 -->
        <aop:pointcut id="p" expression="execution(* com.example.beanlearning.Boy.add(..))"/>

        <!-- 配置切面 表示把boyProxy中的before方法增强到p这个切入点上-->
        <aop:aspect ref="boyProxy">
            <aop:before method="before" pointcut-ref="p"></aop:before>
        </aop:aspect>
    </aop:config>
</beans>
```



