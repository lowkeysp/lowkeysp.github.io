---
title: BGP的路径属性
date: 2020-03-11 11:21:59
tags: ['BGP','数据']
categories: 数据
---

# 属性分类

## 公认属性 well-known
* 公认必遵属性（Well-Known mandatory）
> 所有的BGP实现都必须能识别，且在update报文中必须携带的属性，如Origin，As-Path，Next-hop
* 公认自由决定属性（Well-Known discretionary）
> 所有的BGP实现都必须能识别，但不要求必须包含在update报文中，如Local-Preference，ATOMIC_Aggregate

## 可选属性 Optical

* 可选传递的 (Optical transitive）
> 可以不支持该属性，但即使不支持也应当接受包含该属性的路由并传递给其他对等体，如Community，Aggregator
* 可选非传递的（optical non-transitive）
> 可以不支持该属性，不识别的BGP进程忽略包含这个属性的更新消息并且不传递给其他BGP对等体，如MED，Originator_ID,Cluster_ID,*pre_value

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq03bvwtgj30i30bf782.jpg)

BGP路由前缀指的就是目的地址，如果多个目的地址的路径属性是一样的话，则可以使用一个update报文，BGP路径前缀可以包括好多个目的地址



# Preferred_Value
* 华为BGP私有路径属性，取值范围：0~65535；越大越优先
* 在路由器本地配置，只对本路由器有效，不会传播给任何BGP对等体。
* 本地始发的路由器默认Preferred_Value为0，从其他对等体学到的路由缺省值也是0

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq0amg13zj30fb084mz4.jpg)

R1和R3都给R2宣告了这条路由，但是在宣告的update报文当中，是不携带Preferred_Value属性的，也就是R2都默认为R1和R3的Preferred_Value是0.然后，你可以在R2上对Preferred_Value上进行修改，从而手动优选走R1的路由，当然，这个值也只在R2上有效

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq0gfo6iyj30js08b761.jpg)



# Local_Preference
* 本地优先级，公认自决属性，用于告诉AS中的路由器，哪条路径是离开AS的首先路径
* Local_Preference越大则路径越优，缺省Local_Preference值为100
* 只能发送给IBGP对等体，而不能传递给EBGP对等体

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq0kj270xj30g806vmyq.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq0mhpikgj30j7091dhk.jpg)


* Local_Preference属性只能在IBGP对等体之间传递（除非部署了路由策略否则该属性值在IBGP对等体传递过程中不会丢失），不能在EBGP对等体之间传递
* 如果从EBBGP对等体收到的路由携带了Local_Preference,则会触发Notifacation报文；但是可以在AS边界路由器上使用import方向的策略来修改Local_Preference属性
* 路由器network及重发布到BGP的路由，Local_Preference的缺省值为100，使用`bgp default local-preference`命令可修改缺省Local_Preference值
* BGP路由器在向其EBGP对等体发送BGP路由时，不能携带Local_Preference属性，但是对方会在本地为这条路由赋一个缺省Local_Preference也就是100，然后再将属性传递给自己的IBGP对等体

# AS_PATH
* AS_PATH是公认必遵属性，是BGP路由传递过程中所经过的AS号的列表
* AS_PATH可用于确保路由再EBGP对等体之间传递时的无环，另外也作为路由优选的依据之一。
* BGP路由被通告给EBGP对等体时会再AS_PATH中追加上本地的AS号，通告给IBGP邻居时不修改AS_PATH

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq184x5s5j30hy04z0ud.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq1b5qfjbj30gr0b9adj.jpg)




![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq1pwst8pj30ji09u76f.jpg)

{100，200}是AS_SET，是无序的,长度是1；300与{100，200}是AS_SEQ，是有序的。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq1u7qfgvj30k60d5n4e.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq1wz1br4j30ka0ac0x0.jpg)

# ORIGIN
公认必遵属性

* IGP：标记为i，通过BGP network的路由，也就是起源于IGP的路由，因为BGP network必须保证该网络在路由表中
* EGP: 标记为e，是由EGP这种早期的协议重发布而来
* Incomplete: 标记为？，是从其他渠道学习到的，路由来源不完全（确认该路由来源的信息不完全，重发布的路由，import-route）

# MED
* MED(Multi Exit Discriminator)是可选非传递属性，是一种度量值，用于向外部对等体指出进入AS的首选路径，即当入口有多个时，AS可以使用MED动态地影响其他AS如何进入地路径
* MED属性值越小则BGP路径越优
* MED主要用于在AS之间影响BGP的选路。MED被传递给EBGP对等体后，对等体在其AS内传递路由时，携带该MED值，但不会将该MED值传递给下一个AS。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcq34dpo6vj30nl06g0wr.jpg)

## 注意事项
* 缺省情况下，只比较来自同一对等体AS的BGP路由的MED值，也就是说如果去往同一个目的地的两条路由来自不同的AS，则不进行MED值得比较。
* MED只在直接相邻得AS间影响业务量，而不会跨AS传递
* 一台BGP路由器将路由通告给EBGP对等体时，是否携带MED属性，需要根据以下条件进行判断（不对EBGP对等体使用策略得情况下）：
* * 如果该BGP路由是本地始发（本地通过network或import-route命令引入）的，则缺省携带MED属性发送给EBGP对等体
* * 如果该BGP路由是从其他BGP对等体学习过来的，那么将该路由通告给EBGP对等体时不携带MED。
* 在IBGP对等体之间传递路由时，MED值会被保留并且传递，除非部署了策略，否则MED值在传递过程中不发生改变也不会丢失。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqbihr3twj30np0ccjvo.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqbmt3atlj30od0bygq0.jpg)

# Next_Hop
该属性是一个公认必遵属性，用于指定到达目标网络的下一跳地址。
* 当路由器学习到BGP路由时，需对BGP路由的Next_Hop属性值进行检查，该属性值（IP地址）必须在本地路由可达，如果不可达，则这条BGP路由不可用。
* 在EBGP及IBGP对等体的场景中，Next_Hop的缺省操作是存在差异的。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqbwhsq1qj30nt0ae0v8.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqby8053qj30nx0bgaec.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqc1a9e5fj30mm0b10wl.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqc2n6kqlj30lq0cf77l.jpg)

中间是一个交换机

上图的操作是BGP做了一个优化，也就是这三个路由器通过交换机连接，是在同一个网段的。按照通常情况下，A会把BGP路由传递给C，并且下一跳改成自己的地址，这样，C要访问的B的话得先通过A，再通过B。会发现经过A是多此一举，因为C明明可以直接访问B，因此，如果A发现Next_Hop地址值和要更新的EBGP对等体同属于一个网段的话，则Next_Hop不发生变化

# Community
有了community属性值，可以为不同种类的路由打上不同的community属性值，这些属性值会随着BGP路由更新给其他AS，路由器只需要根据Community属性值来执行差异化的策略即可而不用去关心具体的路由前缀

* 该属性是可选传递属性，是一种路由标记，用于简化路由策略的执行
* 可以将某些路由分配一个特定的community属性值，之后就可以根据community值而不是网络号/掩码信息来抓取路由并执行相应的策略了。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqcjnq94xj30fp06babj.jpg)


## Community属性值
* Community属性值长度为32个比特，也就是四个字节。通常使用两种形式呈现，一是十进制的整数格式，二是十六进制的AA:NN格式，其中AA表示AS号，NN是自定义的编号

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqcra8kvhj30mc07s0uj.jpg)

### no-advertise
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqcsi3qlnj30jl0aj0vs.jpg)

### no-export

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqct6ndfvj30lk0apgp4.jpg)

### no-export-subconfed

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqcvtd3uzj30lk0apgp4.jpg)

# Atomic_Aggregate及aggregator

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqd1400xyj30ki0a30vy.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcqd3ponokj30iy09tn06.jpg)

表明你的汇总路由的路由器和AS，Atomic_Aggregate只是做了一个标记，说明这个是一个汇总路由