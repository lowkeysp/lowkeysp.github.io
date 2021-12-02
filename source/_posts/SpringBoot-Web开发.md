---
title: SpringBoot-Web开发
date: 2021-11-30 14:25:04
tags:
---

# 静态资源访问
默认情况下，静态资源放在classpath（或者ServletContext的根目录）的`/static`或者`/public`或者 `/resources`,或者`/META-INF/resources`下。比如一张图片a.png放在classpath的`/static`目录下，则可以通过`http://localhost:8080/a.png`访问得到。

请求进来，先去找Controller看能否处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面。

默认情况下，使用`/**`进行资源的静态映射。可以对映射进行修改，通过`spring.web.resources.static-locations`,修改资源的静态映射，比如改成`/resources/**`,则配置文件为：
```
# 如果是properties文件
spring.mvc.static-path-pattern=/resources/**
```
或
```yml
# 如果是yaml文件
spring:
  mvc:
    static-path-pattern: "/resources/**"
```
则需要通过`http://localhost:8080/resources/a.png`进行访问

也可以对静态资源的存放位置进行修改，通过`spring.web.resources.static-locations`属性，
比如：
```
spring.web.resources.static-locations=classpath:/test
```
则，需要把静态资源放到classpath的test目录下才能访问到这些静态资源

# 欢迎页
SpringBooot支持static和templated欢迎页。首先，会去寻找配置好的static content位置的index.html，如果没有的话，则会寻找index的template，如果其中有一个存在，则会自动展现欢迎页。






