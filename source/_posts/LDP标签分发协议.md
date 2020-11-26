---
title: LDP标签分发协议
date: 2020-04-01 16:52:03
tags: ['MPLS','数据','LDP']
categories: 数据
---

<meta name="referrer" content="no-referrer" />

# LDP标签分发协议

LDP是专门为标签发布而制定的协议。LDP根据IGP，BGP路由信息通过逐跳的方式建立LSP

## LDP协议概述

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdegydx4clj30iv06t76x.jpg)

* LDP（Label Distribution Protocol，标签分发协议）是一种标签分发协议，广泛应用于MPLS网络，具有配置简单，可提供路由拓扑驱动建立LSP、支持大容量LSP等优点
* LDP是MPLS的一种控制协议。LDP通过Hello报文发现邻居，并基于TCP建立邻居间的会话
* LDP能够动态地为FEC分配标签，并建立LSP（Label Switched Path，标签交换路径）

## LDP对等体及LDP会话

### LDP对等体

* LDO对等体是指相互之间存在LDP会话，使用LDP来交换标签映射信息的两个LSR。LDP对等体通过它们之间的LDP会话获得对方的标签映射。LDP对等体也称为LDP邻居

### LDP会话
* 本地LDP会话（Local LDP Session）：建立会话的两个LSR之间是直连的
* 远程LDP会话（Remote LDP Session）：建立会话的两个LSR之间可以是直连的，也可以是非直连的。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdeh5l8u2vj30jp05i3zg.jpg)

## LDP报文类型

LDP协议主要使用四类报文：
* 发现（Discovery）报文：用于通告和维护网络中LSR的存在，如Hello报文
* 会话（Session）报文：用于建立，维护和终止LDP会对等体之间的会话，如Initialization报文，Keepalive报文
* 通告（Advertisement）报文；用于创建，改变和删除FEC的标签映射
* 通知（Notification）报文：用于提供建议性的报文和差错通知

为了保证LDP报文的可靠发送，除了Discovery报文使用UDP传输外，LDP的其他三种报文都使用TCP传输

## 基本概念

### LSR ID

* 每一台运行MPLS的LSR必须拥有一个域内唯一的LSR ID
* 在华为设备上激活MPLS能力hi前，必须为设备配置MPLS ID（使用全局配置命令 mpls lsr-id）
* LSR ID长度为32bit，与IPv4地址的格式相同
* 通常情况下，我们会选择设备的某个Loopback接口地址作为LSR ID

### LDP ID
* 每一台运行了LDP的LSR除了必须拥有LSR ID，，还必须拥有LDP ID
* LDP ID的长度是48bit，由32bit的LSR ID与16bit的标签空间标识符（Label Space ID）构成
* LDP ID以“LSR ID：标签空间标识”的形式呈现。例如：2.2.2.2：0
* 标签空间标识一般存在两种形态：
* * 值为0，标识基于设备（或基于平台）的标签空间
* * 值为1：标识基于接口的标签空间

### 传输地址
* 两台LSR之间在建立LDP会话之前，需要先建立TCP连接，以便进行LDP协议报文的交换
* 互为邻居的LSR需基于双方的传输地址（Transport Address）建立TCP连接
* 在LDP Hello报文中，包含设备的传输地址，LSR通过Hello报文知晓邻居的传输地址
* 在使用Hello报文发现邻居并且知道了对方的传输地址后，邻居之间就会开始尝试TCP三次握手（基于传输地址），并且交互LDP的初始化报文，标签映射报文等，这些报文都使用双方的传输地址作为源，目的IP地址
* 正如以上所说，传输地址会被用来与邻居建立TCP连接，因此LSR必须拥有到达邻居的传输地址的路由
* 缺省情况下，公网的LDP传输地址等于设备的LSR ID，私网的传输地址等于接口的主IP地址
* 在接口视图下，使用`mpls ldp transport-address`命令，可以修改传输地址

### 本地LDP会话建立过程1：邻居发现与TCP连接建立

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgm15ft6rj30os0ccwka.jpg)

R1发出的Hello报文
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgm6cmvp6j30kd0bkwia.jpg)

### 本地LDP会话建立过程2：LDP会话建立过程

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgm7b77cmj30o80c9jwl.jpg)

R2发出来的Initialization报文

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgna0cqujj30m90cj79q.jpg)

### LDP标签通告过程

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgnbgc6lzj30p30cf0xe.jpg)



### LDP会话状态查看（简要信息）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgndlk4qfj30n80cgaet.jpg)

### LDP会话状态查看（详细信息）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgnfbsv6xj30np0cp42a.jpg)

### LDP邻居查看（简要信息）


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgnh0oqt1j30nq0c1dkl.jpg)

DiscoverySource表示是哪个端口发现的这个邻居

### LDP邻居查看（详细信息）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgniev61pj30nj0cetbw.jpg)


### LDP状态机（RFC5036）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgnj9h0t9j30lv0cftbt.jpg)

## LSR的缺省行为

在华为设备上激活MPLS与LDP后，缺省情况如下：
* Transit LSR只有收到下游（DownStream）的标签映射报文，才会向上游（UpStream）分发标签
* 对于从邻居LSR收到的标签映射，无论邻居LSR是不是自己的下一跳都会保留下来

## LDP基本过程


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgnt0de8oj30p20cqjvw.jpg)


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgns5jyuwj30ot0c80xh.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgnunc9d2j30pi0d242u.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgntt0xmyj30ot0d50x2.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgnxiib82j30os0cgjvy.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgny96lslj30p60d5djv.jpg)

R1的NHLFE里面的Push表示动作，这是压入标签的意思。R1的FIB表的Tunnel-ID表示对应的NHLFE要做的动作是什么

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgo0y6fduj30pc0crwkg.jpg)

## 在MPLS中，运行LDP协议的LSR的操作小结
* LSR首先通过运行IGP协议（例如OSPF，ISIS）来构建路由表，FIB表
* LDP根据相应的模式，为路由表中的路由前缀（FEC）分配标签
* LDP根据相应的模式，将自己为路由前缀分配的标签，通过LDP标签映射报文通告给LDP邻居
* LSR将自己为路由前缀分配的标签，以及LDP邻居为改路由前缀通告的标签存储起来，并形成关联
* 当LSR转发到达目的网络的标签报文时，所使用的出站标签总是下游LDP邻居所通告的标签，此处所指的下游邻居，是设备的路由表中到达该目的网络的下一跳设备



# PHP特性 

## R3没有使用PHP特性之前
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgoa6casoj30mz0cewik.jpg)



## PHP特性 

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgoby96o4j30o20cj7a1.jpg)



![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgog0blryj30p20cr0y4.jpg)


## 显式空标签（应用背景）


如果使用隐性标签的话，会损失一些信息，比如在标签头里面，不光只有标签值，还有比如QoS信息，如果在R3就跳出了，那么就损失了QoS信息
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgohe2mabj30nq0a9die.jpg)


# 显式空标签
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdgokl26lzj30nj0c3td7.jpg)



# label advertise命令小结

* 在MPLS视图下，执行命令`label advertise (explicit-null | implicit-null | non-null)`，配置向倒数第二跳分配的标签
* 根据参数不同，可以配置Egress向倒数第二跳分配不同的标签
* * 缺省情况下是implicit-null，表示支持PHP。Egress向倒数第二跳节点分配隐式空标签，值为3
* * explicit-null，表示不支持PHP。Egress向倒数第二跳节点分配显式空标签，值为0。当需要支持MPLS属性时，可以选用explicit-null
* * non-null，表示不支持PHP。Egress向倒数第二跳节点分配正常标签，即分配的标签值不小于16