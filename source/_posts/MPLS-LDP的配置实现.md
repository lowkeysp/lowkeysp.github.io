---
title: MPLS LDP的配置实现
date: 2020-04-07 15:05:32
tags: ['MPLS','LDP','数据',]
categories: 数据
---

<meta name="referrer" content="no-referrer" />

# LDP的基础配置案例1

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdl7xclzuij30m90ae40z.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdl7zyed8aj30p80c8gpd.jpg)

## 配置检查

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdl8j1a2atj30iq08imzj.jpg)

# 使用MPLS解决BGP路由黑洞问题

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdl99ojar1j30ow0d0grl.jpg)

## BGP路由黑洞问题

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdl9b8jv1aj30kx0biq5m.jpg)



## 解决方案：激活MPLS及LDP
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdl9imvs33j30p90cq44l.jpg)


需要注意的是，下一跳4.4.4.4的路由只能迭代到R2，并不会迭代到LSP，因此，需要使用'route recursive-lookup tunnel'

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdl9lnzxx5j30nv0cndkz.jpg)



## 其他命令

* 配置设备的LDP传输地址：
```
[Quidway-if] mpls ldp transport-address intf.

```
缺省情况下，公网的LDP传输地址等于节点的LSR-ID，私网的传输地址等于接口的主IP地址


* 配置LSP建立的触发策略
```
[Quidway-if] lsp-trigger {all|host|ip-prefix ip-prefix-name | none}
```
缺省情况下，触发策略为host，即32位地址的主机IP路由（不包括接口的32位地址的主机IP路由）触发建立LSP，其他的触发策略如下：
> * 如果触发为all，则所有静态路由和IGP路由触发建立LSP。BGP公网路由不能触发建立LSP
> * 如果触发策略为ip-prefix，则只有通过IP地址前缀列表过滤的FEC项能够触发建立LSP
> * 如果触发策略为none，则不触发建立LSP

host:意思就是如果直连的主机IP地址是32位的，则为这个IP路由建立起一个LSP


* 配置LDP标签策略
一般情况下，LSR会向其上游和下游LDP对等体都分配标签，从而提高了LDP LSP的收敛速度。但是接收所有的标签映射，或者向所有对等体发送标签映射会导致大量LSP的建立，造成资源的浪费。为了减少LSP的数量，节省内存，可配置LDP水平分割策略
* 配置LDP水平分割策略
```
[Quidway-mpls-ldp] outbound peer {peer-id | all} split-horizon
```

为LDP对等体配置水平分割策略，即控制LSR只向其上游LDP对等体分配标签。缺省情况下，没有为LDP对等体配置水平分割策略，即LSR会向其上游和下游LDP对等体都分配标签

