---
title: IS-IS路由渗透
date: 2019-05-21 17:06:51
tags: [路由协议,网络,IS-IS协议]
categories: 网络
---
<meta name="referrer" content="no-referrer" />

# IS-IS路由渗透
通常情况下，Level-1区域内的路由通过Level-1路由器进行管理。所有的Level-2和Level-1-2路由器构成一个连续的骨干区域。Level-1区域必须且只能与骨干区域相连，不同的Level-1区域之间并不相连。

Level-1-2路由器将学习到的Level-1路由信息装进Level-2 LSP，再泛洪LSP给其他Level-2和Level-1-2路由器。因此，Level-1-2和Level-2路由器知道整个IS-IS路由域的路由信息。但是，为了有效减小路由表的规模，在缺省情况下，Level-1-2路由器并不将自己知道的其他Level-1区域以及骨干区域的路由信息通报给它所在的Level-1区域。这样，Level-1路由器将不了解本区域以外的路由信息，可能导致与本区域之外的目的地址通信时无法选择最佳的路由。

为解决上述问题，IS-IS提供了路由渗透功能。通过在Level-1-2路由器上定义ACL（Access Control List）、路由策略、Tag标记等方式，将符合条件的路由筛选出来，实现将其他Level-1区域和骨干区域的**部分路由信息**通报给自己所在的Level-1区域。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g396ra5thkj30e207f74p.jpg)

如上图所示，RouterA发送报文给RouterF，选择的最佳路径应该是RouterA->RouterB->RouterD->RouterE->RouterF。因为这条链路上的cost值为10+10+10+10=40，但在RouterA上查看发送到RouterF的报文选择的路径是：RouterA->RouterC->RouterE->RouterF，其cost值为10+50+10=70，不是RouterA到RouterF的最优路由。

RouterA作为Level-1路由器并不知道本区域外部的路由，那么发往区域外的报文都会选择由最近的Level-1-2路由器产生的缺省路由发送出去，所以会出现RouterA选择次最优路由转发报文的情况

如果分别在Level-1-2路由器RouterC和RouterD上使能路由渗透功能，Aera10中的Level-1路由器就会拥有经这两个Level-1-2路由器通向区域外的路由信息。经过路由计算，选择的转发路径为RouterA->RouterB->RouterD->RouterE->RouterF，即RouterA到RouterF的最优路由。
