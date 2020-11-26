---
title: VLAN间路由互通
date: 2020-02-26 17:46:27
tags: [VLAN,数据,交换]
categories: 数据
---
<meta name="referrer" content="no-referrer" />

# 技术背景
VLAN是广播域。通常两个广播域之间是由路由器连接。广播域之间来往的数据包都是由路由器中继的。因此，VLAN间的通信也需要路由器提供中继服务，这被称作VLAN间路由

# 实现

按照使用路由器采用中继的思想，如果使用物理接口的话，那么有几个VLAN，就需要占用几个路由器的物理接口，但这是不现实的，因为路由器的物理接口资源是非常宝贵的。

因此，提出了子接口的概念

## 通过子接口实现VLAN间路由

在路由器以太网接口上进行“子接口”的划分，与交换机的Trunk链路对接

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcard6588tj30gc07hq5e.jpg)

子接口是在物理接口之上创建的，而且是逻辑接口，且可以配置IP地址，需要指定接口对应的VLAN ID。如图中的GE0/0/0.1和GE0/0/0.2等

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcarimck47j30i00aw42a.jpg)

`arp broadcast enable`可配可不配

这些配置需要在逻辑接口下完成配置 

路由器还可以由防火墙代替


问题：
* 交换机和路由器之间的链路的拥塞可能会比较严重



## 通过vlanif接口实现VLAN间路由


### 什么是VLAN IF接口

VLAN IF是Vlan interface的简称。是一个三层交换机的概念，该交换机是有路由模块的。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcas1353v4j30g50ahq6n.jpg)

### 配置

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcasm5nz0oj307c09g3zo.jpg)

在交换机上，除了需要在和PC连接的两个物理接口上进行配置之外，不需要配置另一个接口了。
```
vlan batch 10 20
# int g0/0/1 
    port link-type access
    port default vlan 10
  int g 0/0/2
    port link-type access
    port default vlan 20    
```
还需要配置

```
interface   vlanif 10
    ip address 192.168.10.254 24
interface vlanif 20
    ip address 192.168.20.254 24
```


### 例子

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcataaaljfj30f108ndhq.jpg)


接入层交换机配置(没有路由模块)
```
vlan batch 10 20
# int g0/0/1 
    port link-type access
    port default vlan 10
  int g 0/0/2
    port link-type access
    port default vlan 20  


  int g0/0/22
    port link-type trunk
    prot allow-pass vlan 10 20

```
核心交换机配置（有路由模块）
```

  int g0/0/22
    port link-type trunk
    prot allow-pass vlan 10 20

  interface   vlanif 10
    ip address 192.168.10.254 24
  interface vlanif 20
    ip address 192.168.20.254 24

# 核心交换机要和路由器通信，那么可以把路由器当作一个终端，配置成vlan id为99的

vlan 99
 int g0/0/24
 port link-type access
 port default vlan 99

interface   vlanif 99
  ip address 192.168.99.1 24
```
路由器配置
```
int g 0/0/0
  ip address 192.168.99.2 24

ip route-static 192.168.10.0 24 192.168.99.1
ip route-static 192.168.200.0 24 192.168.99.1
```

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gcats0fb0qj30fu0aowhb.jpg)

# 参考文献
https://www.bilibili.com/video/av37573316/?spm_id_from=333.788.videocard.0
