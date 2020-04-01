---
title: MPLS概述与静态LSP
date: 2020-04-01 14:01:34
tags: ['数据','MPLS','LSP']
categories: MPLS
---

# MPLS的概述

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gde8kaupm0j30nj0c5dje.jpg)

在IP报文前面会增加一个标签


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gde8mfirq5j30n50cpn2w.jpg)



## 基本术语

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gde8pnqnyjj30mj07ijtj.jpg)

* LSR（Label Switch Router，标签交换路由器）：支持MPLS的路由器（实际上也指支持MPLS的交换机或其他网络设备），该设备能够根据报文的MPLS标签头部对其尽情转发（例如图中的R1，R2及R3）
* MPLS域（Domain）：一系列连续的LSR构成了一个MPLS域，如上图所示
* LER（Label Edge Router）：位于MPLS域边缘，连接其他网络的LSR，例如图中的R1，R3

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gde8vdorowj30ja06e0ue.jpg)

* LSP（Label Switched Path，标签交换路径）：是标签报文穿越MPLS网络到达目的地所走的路径
* LSP实际上是一种类似Tunnel的概念，LSP的建立过程实际上也是报文转发路径中，沿途设备关于特定FEC的标签的确定过程
* LSP可以通过手工的方式建立（静态LSP），或者通过标签分发协议自动建立（动态LSP）
* LSP是单向的，如图所示


* LSP的入口LSR被称为Ingress LSR（入站LSR）
* 位于LSP中间的LSR被称为Transit LSR（中转LSR）
* LSP的出口LSR被称为Egress LSR（出站LSR）
* Ingress LSR及Egress LSR都是LER
* 上图来说，R1是Ingress LSR，R2是Transit LSR，R3是Egress LSR，一条LSP可以有0个，1个或多个Transit LSR，但有且只有一个Ingress LSR和一个Egress LSR

* FEC（Forwarding Equivalence Class，转发等价类）是在转发过程中具有相同处理方式和处理待遇的数据流，可通过IP地址，隧道，Cos等方式来标识一个FEC。例如，在传统的采用最长匹配算法的IP转发中，到同一条路由的所有报文就是一个转发等价类
* 通常在一台设备上，对于一个FEC分配相同的标签
* 属于一个FEC的流量具有相同的转发方式，转发路径和转发待遇。但是并不是所有拥有相同标签的报文都属于一个FEC，因为这些报文的EXP值可能不相同，执行方式可能不同。
* 决定报文属于哪一个FEC的设备是Ingress LSR，因为是它对报文进行分类和压入标签。

## MPLS架构
### 控制平面（Control Plane）
* 控制平面是无连接的，主要功能是负责产生和维护路由信息以及标签信息
* 控制平面IP路由协议（IP Routing Protocol）模块用来传递路由信息，生成路由信息表；标签分发协议（Label Distribution Protocol）模块用来完成标签信息的交换，建立标签转发路径
* 路由信息表RIB（Routing Information Base）：由IP路由协议（IP Routing Protocol）生成，用于选择路由
* 标签分发协议LDP（Label Distribution Protocol）：负责标签的分配，标签转发信息表的建立，标签交换路径的建立，拆除等工作
* 标签信息表LIB（Label Information Base）：由标签分发协议生成，用于管理标签信息

### 数据平面（Data plane）
* 数据平面也称为转发平面，是面向连接的，主要功能是负责普通IP报文的转发以及带MPLS标签报文的转发。
* 数据平面包括IP转发信息表（Forwarding Information Base）和标签转发信息表（Label Forwarding Information Base），当收到普通IP报文时，如果需要按照标签转发，根据标签转发表转发，如果需要转发到IP网络，则去掉标签后根据IP转发表转发
* 转发信息表FIB（Forwarding Information Base）：从RIB提取必要的路由信息生成，负责普通IP报文的转发
* 标签转发信息表LFIB（Label Forwarding Information Base）：简称标签转发表，由标签分发协议在LSR上建立LFIB，负责带MPLS标签报文的转发

## MPLS标签结构

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdeaygxyxbj30mk0ca41x.jpg)

对于 BoS，靠近IP头部的是栈底，靠近二层帧头的是栈顶

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebdexmbyj30m10bawhz.jpg)

### 标签空间

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebgdyzrwj30pa0cugqo.jpg)

### MPLS标签的处理

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebi65455j30oq0akn1e.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebiq2700j30mh0cbwh9.jpg)

### LSP的建立
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebkcuu4fj30o10d00yb.jpg)

### 静态LSP

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebqcc3baj30nz06z419.jpg)


### 静态LSP的建立

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebs6pjz4j30nu0cowjx.jpg)



#### 静态LSP的建立Step 1 部署路由
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebud5ssuj30o40bg78e.jpg)

#### 静态LSP的建立Step 2  在设备上激活MPLS
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdebxwnxsej30m70bsjwb.jpg)

#### 静态LSP的建立Step 3  配置静态LSP

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdec0515pmj30nz0cojvd.jpg)

注：1to4是自己起的名字 4.4.4.0 24是目的地址和掩码 nexthop是下一跳，需要注意的是，你的目的地址/掩码和下一跳必须和路由表中一致


反向： 
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdec3kn42vj30o60cqjvd.jpg)

#### 静态LSP的建立Step 34  查看与验证

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdec4wuzcxj30nv0c8q7c.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdec67fdc2j30nx0cqn1c.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdec6ur9foj30ne0cidjl.jpg)


#### 静态LSP配置注意事项

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdec7ej06zj30me0chgp5.jpg)



