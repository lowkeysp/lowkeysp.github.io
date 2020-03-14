---
title: BGP的路由策略
date: 2020-03-12 10:28:48
tags: ['BGP','数据','路由策略']
categories: 数据
---

# BGP路由自动汇总

* BGP支持路由自动汇总功能，该功能缺省关闭，可以在BGP配置视图中使用'summary automatic'命令开启
* BGP路由自动汇总功能只对本地采用'import-route'命令注入的BGP路由有效
* 开启该功能后，'import-route'注入的BGP路由会按主类网络进行汇总（按照IP地址所属类别的缺省网络掩码进行计算），所产生的汇总路由会发布到BGP中，而明细路由则会被抑制
* BGP路由自动汇总功能对汇总路由的把控较差，只能够将路由汇总成主类网络路由，无法做到精细化的掩码控制

>主类网络解释：
>
>192.168.1.0/24， 这是一个C类网络（/24），则主类网络是（192.168.1.0/24）
>
>172.16.1.0/24，这是一个B类网络（/16），则主类网络是（172.16.0.0/16）
>
>10.1.1.0/30，这是一个A类网络（/8），则主类网络是（10.0.0.0/8）


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqy6c45jdj30ld0c4n15.jpg)

# BGP手工路由汇总

BGP存在两种手工汇总方案

方案一：（非主流）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcr9nan9txj30hk08q773.jpg)

方案二：

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcr9poh79xj30is07m0us.jpg)


## aggregate
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrfxm14cmj30gs08vdj4.jpg)

由于汇总路由不带任何路径属性，那么将汇总路由传递给R1或者R2时，有可能会产生路由环路

## aggregate detail-suppressed
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrfz20msxj30m00axgq8.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrg1is1z6j30lh0bmael.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrg2d55hgj30lo0c5af2.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrg3b17qsj30o30bxgss.jpg)

## aggregate detail-suppressed as-set

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrg4zkexvj30le0bln2h.jpg)

加了as-set会防止环路，AS_PATH是 300，{100，200}

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrg7jw3phj30og0c0q7u.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrga5bsb6j30nf0brtd6.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrgaufj9xj30ma0bw426.jpg)


## aggregate suppress-policy

aggregate detail-suppressed 是一刀切的做法，也就是把所有的明细路由都抑制掉，而suppress-policy则是选择性的抑制某些路由

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrgch456rj30m30bnjvq.jpg)

* Aggregate命令的suppress-policy关键字用于通告汇总路由以及选定的明细路由（或者说只抑制特定的明细路由）
* 如果在aggregate命令后关联datail-suppressed关键字，则所有明细都将被抑制，如果需要在产生汇总路由的同时通告特定的明细路由，则可关联suppress-policy关键字，在其后调用定义好的route-policy，被route-policy permit的路由将被过滤，其他路由被放行
* 抑制列表虽然调用route-policy，但route-policy只用于匹配（if-match），不能用于设置属性（不能用apply命令）

## aggregate attribute-policy

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrgmd2w8mj30m30c6n1a.jpg)

## aggregate origin-policy

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrgueek9kj30ns0bw78f.jpg)

* Origin-policy关键字可调用一个已经定义好的route-policy，在该route-policy中使用if-match语句匹配明细路由。如此一来，aggregate命令产生的汇总路由相当于是为上述明细路由而产生，只要这些明细路由中，至少有一条路由处于活跃状态，那么汇总路由就会被通告，如果route-policy所匹配的所有明细路由都失效，那么汇总路由也随之消失。

* 需注意，'aggregate 172.16.0.0 16 detail-suppressed origin-policy ori'这条命令产生的汇总路由172.16.0.0/16是与ori中所匹配的路由强关联的，如果存在172.16.0.0/16的一些子网，而这些子网并没有被ori所匹配，那么这些子网将被认为与汇总路由172.16.0.0/16的生成无关（即使从IP角度，它们确实是该汇总路由的子网），因此在本例中即使使用了detail-suppressed关键字，却并没有抑制掉172.16.0.0/24及172.16.11.0/24这两条明细路由

（如果只是'aggregate 172.16.0.0 16 detail-suppressed'，那么对这四条路由都有效，只有这四条明细路由都失效了，汇总路由才失效，否则，只要还有一条生效，汇总路由都还存在）

# 正则表达式

 原子字符：

 ```
 .    匹配任何单个的字符，包括空格
 ^     一个字符串的开始
 $     一个字符串的结束
 _     下划线，匹配任意的一个分隔符，如^,$,空格,tab,逗号,{,}
 |     管道符，逻辑或
 \     转义符，用来将紧跟其后的控制字符转变为普通字符 
 ```


乘法字符：
```
*    匹配前面字符0次或多次出现
+    匹配前面字符1次或多次出现
？   匹配前面字符的0次或1次出现
```

范围字符：
```
[]    表示一个范围，只匹配包含在范围内的字符之一。可以在一个[]的开始使用^来排除范围内的所有字符，也可以使用短横线-来指定一个区间
```

# as-path-filter

使用正则表达式匹配AS_PATH

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrhsv72wzj30f409b77j.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrhtte4loj30ka09q0u6.jpg)

其中，443是一个正则表达式

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrhvwdqzgj30hn09dtb1.jpg)

## 配置命令

配置as-path-filter
```
[Router] ip as-path-filter 1 {permit|deny} regex
```


```
[Router-bgp] peer xxxx as-path-filter 1 (import|export)
```
关键as-path-filter到BGP Peer，起到路由过滤作用


## as-path-filter验证及查看

查看配置的as-path access-list
```
[Router]display ip as-path-filter
```

显示BGP表中所有AS_PATH被该正则表达式匹配的路由,可用来验证正则表达式写的对不对
```
[Router] display bgp routing-table regular-expression
```

显示BGP表中所有被该as-path-filter匹配的路由
```
[Router] display bgp routing-table as-path-filter
```


## 使用as-path-filter匹配路由

应用1：
```
ip as-path-filter 1 permit ^100$

bgp 300
    peer 10.1.34.4 as-path-filter 1 export
```

应用2：
```
ip as-path-filter 1 permit ^100$

route-policy RP permit node 10
    if-match as-path-filter 1
    apply local-preference 100

bgp 300
    peer 10.1.34.4 route-policy RP import

```

## 配置实例1

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrib2ahehj30mm09pae2.jpg)

```
ip as-path-filter 1 permit .* 表示匹配任何，加起来就是拒绝600后还得匹配任意，不然都拒绝了
```

## 配置实例2

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcrid1hku1j30mc0awn2l.jpg)

其中，需要增加一个node 20.不然的话node 10只是匹配了起始是600的，其他的路由默认是deny的，所以增加了一个空的node 20，默认都是允许的

```
peer 10.1.23.3 advertise-community
```
表示要发送这个community属性 否则默认是不发送的，因为你在route-policy的时候 将community设置了no-export。但是你是想让R3不发送，所以得加这条命令




# Community

## 为BGP路由设置Community

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctn7vf2r5j30hk0bc43u.jpg)


每一跳都需要配置'advertise-community'，否则community属性不会跟着路由传过去

## 为BGP路由追加Community

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctnjpc7s5j30hz0amafd.jpg)

一个路由可以携带多个community属性值，

## 使用ip community-filter匹配community属性

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctnq3jc39j30it0awq69.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcto0r0gu8j30i1088q4w.jpg)

## 使用ip community-filter匹配团体属性（严格匹配）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcto30mwxnj30it096412.jpg)

## 删除一个community值

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcto5ap81qj30j50b6ae2.jpg)

## 删除多个community值

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcto8ir9s0j30he0b342i.jpg)

# ip-prefix
## 在BGP中使用IP-Prefix进行路由过滤 示例1

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctoi7p48sj30hu0akwgw.jpg)

## 在BGP中使用IP-Prefix进行路由过滤 示例2

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctojgqh0pj30i20amjtu.jpg)

# Filter-Policy

## Filter-Policy配置示例1

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctokoxchpj30gx0ar40w.jpg)

## Filter-Policy配置示例2

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctomka2p8j30gy0apgnx.jpg)


## Filter-Policy配置示例2 关联协议来源


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctoo0fannj30il0b5gob.jpg)

`filter-policy ip-prefix 2 export direct`指的是将直连路由export出去，所要所用的过滤策略

# route-policy

## Network命令关联route-policy

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctowgafldj30mk0arq6t.jpg)

## Peer命令关联route-policy

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctozfo2bsj30mc0avn17.jpg)

## 使用route-policy过滤路由

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gctp0aqtx3j30mq0af771.jpg)
