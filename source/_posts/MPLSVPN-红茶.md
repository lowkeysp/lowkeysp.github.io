---
title: MPLSVPN-红茶
date: 2020-07-09 20:29:24
tags: ['MPLS','VPN']
categories: 数据
---



# VRF MODEL

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggkzk3a036j311l0k4k29.jpg)

一个VRF可以有多个接口，一个接口只能属于一个VRF


# VRF 和 MP-BGP的配置
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggl09m806bj31070k3gym.jpg)

* 在PE上创建VRF，将PE-CE的接口放入VRF
```
ip vrf cisco
 rd 1:1
 route-target export 234:2       # 本地的RT export
 route-target import 234:4       # 匹配PE2所配置的RT export

interface XX
 ip vrf forwarding cisco         #将该接口放入VRF cisco
 ip address XXXX XXXX

```
* PE配置MP-MGP
```
[cisco] router bgp 234
[cisco] bgp router-id 2.2.2.2
[cisco] neighbor 4.4.4.4 remote 234
[cisco] neighbor 4.4.4.4 update-source loopback0
[cisco] address-family vpnv4
[cisco] neighbor 4.4.4.4 activate
[cisco] neighbor 4.4.4.4 send-community extended



#查看MP-BGP的邻居
[cisco] show ip bgp vpnv4 all summary
```
* 完成PE-CE之间路由的重发布
```

```