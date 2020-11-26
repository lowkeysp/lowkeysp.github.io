---
title: MMPLS转发过程
date: 2020-04-07 16:30:47
tags: ['数据','MPLS','LSP']
categories: 数据
---

<meta name="referrer" content="no-referrer" />

# MPLS详细转发过程

* NHLFE（Next Hop Label Forwarding Entry，下一跳标签转发表项）用于指导MPLS报文的转发。
> NHLFE包括：Tunnel ID，出接口，下一跳，出标签，标签操作类型等信息。
* FTN （FEC-to-NHLFE,FEC到一组NHLFE的映射）
> 通过查看FIB表中Tunnel ID值不为0x0的表项，能够获得FTN的详细信息。FTN只在Ingress存在。
* ILM（Incoming Label Map，入标签到一组下一跳标签转发表项的映射）
> ILM包括：Tunnel ID，入标签，入接口，标签操作类型等信息
>
> ILM在Transit LSR的作用是将标签和NHLFE绑定。通过标签索引ILM表，就相当于使用目的IP地址查询FIB，能够得到所有的标签转发信息。

* Tunnel ID
> 为了给使用隧道的上层应用（如VPN，路由管理）提供统一的接口，系统自动为隧道分配了一个ID，也称为Tunnel ID。该Tunnel ID的长度为32bit，只是本地有效。这是设备为各种隧道所分配的一个ID。在MPLS中，Tunnel ID还用于将FIB，ILM及NHLFE进行关联

## 在报文转发过程中

在Ingress LSR，通过查询FIB表和NHLFE表指导报文的转发

* 当IP报文进入MPLS域时，首先查看FIB表，检查目的IP地址对应的Tunnel ID值是否为0x0.
> * 如果Tunnel ID值为0x0，则进入正常的IP转发流程。
> * 如果Tunnel ID值不为0x0，则进入MPLS转发流程

在Transit LSR，通过查询ILM表和NHLFE表指导MPLS报文的换法。

在Egress LSR,通过查询ILM表指导MPLS报文的转发或查询路由表指导IP报文转发



![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdldk42f64j30o40cl0wr.jpg)





![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdldm58w87j30nm0codkt.jpg)

R2是怎么知道这是一个标签报文呢？在L2数据链路层的报文头里面会有标识，表明后面跟的是标签还是IP报文头


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdldomw3kej30on0ckdkg.jpg)


 
