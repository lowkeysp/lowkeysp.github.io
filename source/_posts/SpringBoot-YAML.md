---
title: SpringBoot-YAML
date: 2021-11-26 09:48:27
tags:
---
# 基本语法
* 大小写敏感
* 使用缩进表示层级关系
* 缩进不允许使用tab，只允许空格
* 缩进的空格数不重要，只要相同层级的元素左对齐即可
* `#`表示注解
* 字符串无需加引号，如果要加，''与""表示字符串内容，会被转义/不转义，比如，'a \n b'，\n会被转义，而"a \n b",\n 不会被转义

# 数据类型
YAML 支持以下几种数据类型：
* 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
* 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
* 纯量（scalars）：单个的、不可再分的值

# YAML 对象
对象键值对使用冒号结构表示 key: value，冒号后面要加一个空格。也可以使用`key:{key1: value1, key2: value2, ...}`,还可以使用缩进表示层级关系；
```yml
key: 
    child-key: value
    child-key2: value2
```

较为复杂的对象格式，可以使用问号加一个空格代表一个复杂的 key，配合一个冒号加一个空格代表一个 value：
```yml
?  
    - complexkey1
    - complexkey2
:
    - complexvalue1
    - complexvalue2
```
意思即对象的属性是一个数组`[complexkey1,complexkey2]`，对应的值也是一个数组 `[complexvalue1,complexvalue2]`

# YAML 数组
以`-`开头的行表示构成一个数组：
```yml
- A
- B
- C
```

YAML 支持多维数组，可以使用行内表示：
```yml
key: [value1, value2, ...]
```
数据结构的子成员是一个数组，则可以在该项下面缩进一个空格。
```yml
-
 - A
 - B
 - C
```
一个相对复杂的例子：
```yml
companies:
    -
        id: 1
        name: company1
        price: 200W
    -
        id: 2
        name: company2
        price: 500W
```
意思是 companies 属性是一个数组，每一个数组元素又是由 id、name、price 三个属性构成。

数组也可以使用流式(flow)的方式表示：
```yml
companies: [{id: 1,name: company1,price: 200W},{id: 2,name: company2,price: 500W}]
```
# 复合结构
数组和对象可以构成复合结构，例：
```yml
languages:
  - Ruby
  - Perl
  - Python 
websites:
  YAML: yaml.org 
  Ruby: ruby-lang.org 
  Python: python.org 
  Perl: use.perl.org
```
转换为 json 为：
```yml
{ 
  languages: [ 'Ruby', 'Perl', 'Python'],
  websites: {
    YAML: 'yaml.org',
    Ruby: 'ruby-lang.org',
    Python: 'python.org',
    Perl: 'use.perl.org' 
  } 
}
```
# 纯量
纯量是最基本的，不可再分的值，包括：
* 字符串
* 布尔值
* 整数
* 浮点数
* Null
* 时间
* 日期
使用一个例子来快速了解纯量的基本使用：
```yml
boolean: 
    - TRUE  #true,True都可以
    - FALSE  #false，False都可以
float:
    - 3.14
    - 6.8523015e+5  #可以使用科学计数法
int:
    - 123
    - 0b1010_0111_0100_1010_1110    #二进制表示
null:
    nodeName: 'node'
    parent: ~  #使用~表示null
string:
    - 哈哈
    - 'Hello world'  #可以使用双引号或者单引号包裹特殊字符
    - newline
      newline2    #字符串可以拆成多行，每一行会被转化成一个空格
date:
    - 2018-02-17    #日期必须使用ISO 8601格式，即yyyy-MM-dd
datetime: 
    -  2018-02-17T15:02:31+08:00    #时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
```
# 引用
`&` 锚点和 `*` 别名，可以用来引用:
```yml
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
```
相当于:
```yml
defaults:
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  adapter:  postgres
  host:     localhost

test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
```
`&` 用来建立锚点（defaults），`<<` 表示合并到当前数据，`*` 用来引用锚点。

下面是另一个例子:
```yml
- &showell Steve 
- Clark 
- Brian 
- Oren 
- *showell 
```
转为 JavaScript 代码如下:
```JavaScript
[ 'Steve', 'Clark', 'Brian', 'Oren', 'Steve' ]
```

# 配置提示
自定义的类和配置文件绑定一般没有提示，可以使用`spring-boot-configuration-processor`实现自动提示，这样，在编写yaml文件时，可以自动提示出对象的属性等
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```
由于这个依赖只是针对开发便捷而导入的，实际上对于应用没有任何作用，所以在打包的时候，不需要把这个依赖打包进来，可以使用下面让这个依赖在打包的时候不用打包进来
```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

