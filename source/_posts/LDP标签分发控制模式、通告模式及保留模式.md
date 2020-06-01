---
title: LDP标签分发控制模式、通告模式及保留模式
date: 2020-04-07 18:32:17
tags: ['LDP','MPLS','数据']
categories: 数据
---


# 标签通告模式 
标签通告模式 Label Advertisement Mode:在MPLS体系中，由下游LSR决定将标签分配给特定FEC，再通知上游LSR，即标签由下游指定，标签的分配按从下游到上游的方向分发。标签发布方式有两种方式。具有标签分发邻接关系的上游LSR和下游LSR必须对使用的标签发布方式达成一致。
* 下游自主 Downstream Unsolicited（设备默认采用该方式）：不需要上游给自己发请求，主动给上游发标签
* 下游按需 Downstream On Demand ：不主动发，上游发请求之后才会给上游发标签


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfivix33j30od0a3tar.jpg)

## 下游自主方式

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfjpi9ynj30n90ch78s.jpg)


## 下游按需方式

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfkq3808j30nh0baq6d.jpg)




# 标签分发控制模式 

标签分发控制模式 Label Distribution Control Mode:标签分发控制方式是指在LSP的建立过程中，LSR分配标签时采用的处理方式
* 独立控制 Independent Control ：不需要等到下游给自己标签，然后捆绑后发给上游自己的标签
* 有序控制 Ordered Control（设备默认采用该方式）：需要等到下游给自己标签，然后捆绑后将自己的标签才发给上游


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlflv13ipj30nz06vdia.jpg)

## 独立控制

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfmxln8rj30or0c9jvr.jpg)

## 有序控制

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfnwuf5yj30od0chgro.jpg)







# 标签保留方式 
标签保留方式 Label Retention：标签保留方式是指LSR对收到的，但目前暂时不需要的标签映射的处理方式
* 自由模式 Liberal Retention （设备默认采用该方式）：会把所有的标签映射都在本地保留起来
* 保守模式 Conservative Retention ：只会保存使用的标签值

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfoiimkuj30no0bngoz.jpg)

## 自由模式

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfr8vckdj30nt0bh77p.jpg)

## 保守模式

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gdlfrr0o2lj30lb0c5n0x.jpg)

# 华为设备支持的组合模式

目前华为设备支持如下组合方式：

* 第一种：下游自主方式 + 有序控制 + 自由模式 （该方式为缺省方式）
* 第二种： 下游按需方式 + 有序控制  +  保守模式

# 配置LDP标签发布方式

在接口视图下：

配置标签发布方式
```
mpls ldp advertisement {dod | du}

```
* 缺省情况下，标签发布模式为下游自主标签分发DU
* 邻居之间存在多链路的时候，所有接口的标签发布方式必须相同






