---
title: MPLS VPN
date: 2020-07-08 11:47:19
tags: ['MPLS','VPN']
categories: MPLS
---


* CE（Customer Edge）设备：企业用户的网络设备
* PE（Provider Edge）设备：运营商提供边缘设备
* P（Provider）设备：运营商骨干设备

# MPLS VPN产生的原因
两个客户的VPN存在相同的地址空间（私网地址），传统VPN网络结构中的设备无法区分客户重叠的路由信息

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjjllqe9ij31rk0tk7wh.jpg)

解决VPN客户地址空间重叠问题需要解决上述三个问题。

 
 解决方案：

解决问题1：

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjjxhvogej31o10jsnmc.jpg)

 在共享PE设备上使用VRF（VPN Routing and Forwarding VPN路由转发）技术将重叠的路由隔离：每个VPN的路由放入自己对应的VPN Routing Table中

 PE设备在维护多个VPN Routing Table时，同时还维护一个公网的路由表

 PE设备除有多个不同实例的VPN路由表之外，还存在公网路由表（全局路由表）；每个PE设备对应一个或多个VPN实例，与PE设备接口进行一一绑定，每个VRF都需要一个VPN Instance，PE设备多个不同VPN实例相互隔离



解决问题3：

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjk5rh1esj31p00w1hdt.jpg)


将VPN路由发布到全局路由表之前，使用一个全局唯一的标识和路由绑定，以区分冲突的私网路由。这个标识被称为RD（Route Distinguisher）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjken5op4j31p60s47wh.jpg)

但是这种还是存在一定的问题，比如两个分部和一个总部，这个RD到底是应该用1：1还是2：2，似乎都不行。

因此，RD并不能完美解决这一问题，因此又引入了RT来解决

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjkio84bbj31l70sp4qp.jpg)

RT属性用于将路由正确引入VPN，有两类VPN Target属性，Import Target和Export Target，分别用于VPN路由的导出与导入。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjkydnv78j31su0wfhdt.jpg)

使用RT实现本端与对端的路由正确引入VPN，原则如下：
* 本端的Export Target = 对端的Import Target，本端的Import Target = 对端的Export Target

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjl3aglmaj31wz0ze7wh.jpg)

因为数据包没有携带任何标识，所以在ICMP的数据包到达PE1时，PE1并不知道该查找哪个VPN的路由表找到正确的目的地址。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjl8cueetj31z5167u0x.jpg)

使用标签嵌套解决数据转发过程中冲突路由的查找问题

# MPLS VPN的工作过程

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjlic7kumj31vl0y24qp.jpg)

MPLS VPN的工作过程分为两部分：
* MPLS VPN路由的传递过程
* MPLS VPN数据的转发过程

## CE与PE之间的路由交换

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjlkm2bdfj31ur0y77wh.jpg)

PE与CE之间可以通过静态路由协议交换路由信息，也可以通过动态路由协议交换路由信息

## VPN路由注入MP-BGP(多协议BGP)的过程
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjlmvg1hdj31qp0s8b29.jpg)

VRF中的IPv4路由被添加上RD，RT与标签等信息称为VPN-IPv4的路由放入到MP-BGP的路由表中，并通过MP-BGP协议在PE设备之间交换路由信息

MP-BGP是多协议BGP，可以用来传递VPN-IPv4路由，而普通的BGP协议是不能传递VPN-IPv4路由的

PE1设备与P设备之间是通过MPLS来转发的。

## 公网标签的分配过程

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjlykjlnsj31z20web29.jpg)

* MPLS协议在运营商网络分配公网标签，建立标签隧道，实现私网数据在公网上的转发
* PE之间运行的MP-BGP协议为VPN路由分配私网标签，PE设备根据私网标签将数据正确转发给相应的VPN


## PE设备到CE设备的数据转发

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjm207ndej31un12ie81.jpg)

PE1收到剥离公网标签的数据包后，根据私网标签查找转发数据包的下一跳，将数据包正确发给相应的VPN客户


# MPLS VPN配置


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggjmqk5b7yj325t1b0npg.jpg)

关键配置：

PE1上面的配置
```
#创建VPN实例
[PE1] ip vpn-instance vpn1
#使用ipv地址簇
[PE1-vpn-instance-vpn1] ipv4-family
 

[PE1] ip vpn-instance vpn2
[PE1-vpn-instance-vpn2] ipv4-family


# 绑定IP地址,如果直接给端口绑定的话，由于地址一样，会发生冲突
[PE1] interface GigabitEthernet0/0/1
[PE1-GigabitEthernet0/0/1]ip binding vpn-instance vpn1
[PE1-GigabitEthernet0/0/1] ip address 192.168.1.1 24

[PE1] interface GigabitEthernet0/0/2
[PE1-GigabitEthernet0/0/2]ip binding vpn-instance vpn2
[PE1-GigabitEthernet0/0/2] ip address 192.168.1.1 24

# 配置RD
[PE1] ip vpn-instance vpn1
[PE1-vpn-instance-vpn1] route-distinguisher RD值
#配置RT
[PE1-vpn-instance-vpn1] vpn-target RT值 export-extcommunity
[PE1-vpn-instance-vpn1] vpn-target RT值 import-extcommunity


```

配置MP-BGP

在PE2上
```
[PE2] bgp 300
[PE2-BGP] peer 3.3.3.3 as 300
[PE2-BGP] peer 3.3.3.3 connected loopback0
[PE2-BGP] peer 3.3.3.3  next-hop-local
#支持vpnv4簇
[PE2-BGP] ipv4-family vpnv4
[PE2-BGP-af-vpnv4] peer 3.3.3.3 enable

```

在PE2上配置BGP与CE3所在的AS 400建立BGP邻居

```
CE3建立BGP邻居的命令和普通建立是一样的，PE2有些不同

[PE2] bgp 300
[PE2-BGP] ipv4-family vpn-instance vpn3
[PE2-BGP-vpn3] peer 192.168.2.2 as-number 400


```






# VRF MODEL

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggkzk3a036j311l0k4k29.jpg)

