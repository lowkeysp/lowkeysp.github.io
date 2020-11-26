---
title: BGP的选路规则
date: 2020-03-14 19:49:03
tags: ['数据','BGP']
categories: 数据
---

<meta name="referrer" content="no-referrer" />

# 介绍

BGP的选路规则与BGP路径属性及路由策略息息相关，它们使得BGP拥有了强大的路由操控能力。

在BGP路由表中的路由必须首先是可用的（Valid），可盈的路由表项行首存在“*”号，可用意味着该BGP路由的Next_Hop是路由可达的。如果BGP路由不可用，则不会被优选

# 优选规则概览

不同的厂商在优选规则上存在细微差异


1. 优先具有最大Preferred-Value的路由
2. 优选具有最大Local-Preference的路由
3. 优选起源于本地的路由
4. 优选AS_PATH最短的路由
5. Origin（IGP > EGP > Incomplete）
6. 优选MED最小的路由
7. 优选EBGP对等体所通告的路由
8. 优选到Next_Hop的IGP度量值最小的路由
9. BGP路由分载分担
10. 优选Cluster_List最短的路由
11. 优选Router-ID最小的BGP对等体发来的路由
12. 优选Peer-IP地址最小的对等体发来的路由


# 优先具有最大Preferred-Value的路由

* 华为私有的路径属性，相当于路由的权重值，取值范围：0~65535；该值越大，则路由越优先。
* Preferred-Value只能在路由器本地配置，而且只影响本设备的路由优选。该属性不会传播给任何BGP对等体。
* 路由器本地始发的BGO路由默认的Preferred-Value未0，从其他BGP对等体学习到的路由默认Preferred-Value也为0


## 修改从特定对等体收到的“所有路由”的Preferred-Value
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdbzey8xj2j30lb0d043f.jpg)


## 使用route-policy修改Preferred-Value
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdbzgn89abj30mh0ctteh.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdbzl8tvmlj30nk0bfwiq.jpg)

# 优选具有最大Local-Preference的路由

## 通过route-policy修改Local-Preference

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdbzt045mbj30mf0ccgr0.jpg)

# 优选起源于本地的路由

* 在其他条件相同的情况，优先本地生成的路由（本地生成的路由优先级高于从邻居学来的路由）
* 本地生成的路由包括通过network或import-route命令引入的路由，手工汇总路由和自动汇总路由。这些本地生成的路由之间的优选如下：
> 1. 优先汇总路由（汇总路由优先级高于非汇总路由）
> 2. 通过aggregate命令生成的手动汇总路由的优先级高于通过summary automatic命令生成的自动汇总路由
> 3. 通过network命令引入的路由优先级高于import-route命令引入的路由


# 优选AS_PATH最短的路由

## 通过route-policy修改AS_PATH属性

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc07n8d36j30n20cqwk9.jpg)

由于AS_PATH属性对于防环有着重要的作用，因此AS_PATH这个属性尽量不要多修改


* 使用route-policy修改BGP路由的AS_PATH:
> * apply as-path XXX additive   在已有AS_PATH基础上追加XXX
> * apply as-path XXX overwrite   将已有AS_PATH值替换（覆盖）成XXX
> * apply as-path none overwrite   清空路由的AS_PATH属性

* 使用route-policy修改BGP路由的AS_PATH时，可以在EBGP对等体之间改变EBGP路由的AS_PATH属性，从而影响BGP路由的优选，在华为路由器上，在IBGP对等体之间，也可以使用route-policy修改BGP路由的AS_PATH。无论哪种场景，改变BGP路由的AS_PATH都必须十分谨慎。
* Bestroute as-path-ignore命令用来配置BGP在选择最优路径时忽略AS路径属性。配置该命令后，BGP将不比较AS_PATH路径的长度。缺省情况下，长度更小者优

## AS_PATH选路规则扩展1

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc0lktnzjj30ov0brdln.jpg)

其中，{100，200}是一个AS_SET，长度为1.

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc0o1tu89j30nl0bx0xl.jpg)

## AS_PATH选路规则扩展2

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc0omz13vj30nt0cwn0z.jpg)



# Origin（IGP > EGP > Incomplete）

* igp：通过BGP network的路由，也就是起源于IGP的路由，Origin为igp。因为BGP network必须保证该网络在路由表中
* egp：如果BGP路由是由EGP这种早期的协议重发布来的，那么origin是egp
* incomplete：通过import命令，从其他协议引入到BGP的路由，其origin为incomplete（确定该路由来源的信息不完全）

## 使用route-policy修改路由Origin属性值

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc0wq2s0aj30gp09bdje.jpg)



# 优选MED最小的路由

* MED（Multi Exit Discriminator）是可选非传递属性，是一种度量值，用于向外部对等体指出进入本AS的首选路径，即当进入本AS的入口有多个时，AS可以使用MED动态地影响其他AS选择进入地路径
* MED属性值越小则BGP路径越优
* MED主要用于在AS之间影响BGP的选路。MED被传递给EBGP对等体后，对等体在其AS内传递路由时，携带该MED值，但将路由传递给其EBGP对等体时，缺省不会携带MED属性

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc11bpkmnj30nc06kjv0.jpg)

## 使用route-policy修改路由的MED值

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc1byqrwsj30lz0cbag0.jpg)

**（上图其实有一个问题，就是MED值通常只比较来自同一AS的情况）**


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc1d7sfhjj30nz0b2grk.jpg)



# 优选EBGP对等体所通告的路由（相对于IBGP对等体）


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc1gr50hhj30oe0bsdkr.jpg)



# 优选到Next_Hop的IGP度量值最小的路由

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc1mzq100j30ov0bntde.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc1rfpdl4j30oe0byn1t.jpg)

R5学到的这两条BGP路由的next-hop都是3.3.3.3,因此比较next-hop的IGP cost值是没有意义的，是无法做出比较的，只能根据后面的选路规则来比较

# BGP路由分载分担


## 没有配置 BGP路由负载分担时

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc2z0k9dfj30nt0cjq7s.jpg)

* 在大型网络中，到达同一目的地通常会存在多条有效BGP路由，设备只会优选一条最优的BGP路由，将改路由加载到路由表中使用，并且只将最优路由发布给对等体，这一特点往往会造成很多流量负载不均衡的情况。通过配置BGP负载分担，可以使得设备同时将多条等代价的BGP路由加载到路由表，实现流量负载均衡，减少网络拥塞。
* 值得注意的是，尽管配置了BGP负载分担，设备依然只会在多条到达同一目的地的BGP路由中优选一条路由，并只将这条路由通告给其他对等体
* 形成BGP等价负载分担的条件是“BGP路由优选规则”的1至8条规则中需要比较的属性完全相同，例如相同的Preferred_Value，Local_Preference,AS_PATH,MED,Origin,到达Next_Hop的IGP度量值，路由类型（IBGP，EBGP）等
* 如果实现了BGP负载分担，则不论是否配置了peer next-hop-local命令，本地设备向IBGP对等体组发布路由时都先将下一跳地址改变为自身地址。
* 在公网中到达同一目的地的路由形成负载分担时，系统会首先判断最佳路由的类型。若最优路由为IBGP路由则只是IBGP路由参与负载分担，若最优路由为EBGP路由则只是EBGP路由参加负载分担。即公网中到达同一目的地的IBGP和EBGP路由不能形成负载分担
* 如果到达目的地址存在多条路由，但是这些路由分别经过了不同的AS缺省情况下，这些路由不能形成负载分担，如果用户需要这些路由参与负载分担，就可以执行load-balancing as-path-ignore命令。配置load-balancing as-path-ignore命令后会改变路由参与负载分担的条件，路由形成负载分担时不再比较AS_PATH属性，配置时需要慎重考虑
* load-balancing as-path-ignore命令和bestroute as-path-ignore命令互斥，不能同时使能

## 配置了BGP路由负载分担之后1

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc3jlw20sj30oa0cmtg2.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc3me9uozj30nq0cm0xi.jpg)


## 配置了BGP路由负载分担之后2

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc3n6dq3hj30nb0cbgqk.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc3nyz4alj30n10ctaet.jpg)

# 优选Cluster_List最短的路由

## 优选Cluster_List最短的路由 案例1

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc3qfwy3kj30nz0cnaeq.jpg)


## 优选Cluster_List最短的路由 案例2

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc3zpkcstj30p90b7n15.jpg)


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc40xoj22j30jz0cf0wa.jpg)




# 优选Router-ID最小的BGP对等体发来的路由

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc42eica2j30lt0ckq6l.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc45ka84lj30oe0bxjwp.jpg)


# 优选Peer-IP地址最小的对等体发来的路由

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdc4fsbfq4j30o60cojy9.jpg)


















