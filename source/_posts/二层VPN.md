---
title: 二层VPN
date: 2020-08-31 08:43:18
tags: ['数通','VPN']
categories: 数通
---

<meta name="referrer" content="no-referrer" />


MPLS L2VPN提供基于MPLS网络的二层VPN服务，包括**点到点的VPWS业务**（Virtual Private Wire Service）跟**点到多点的VPLS业务**（Virtual Private LAN Service）


# 简介


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gi9pfws5u8j31810je7fn.jpg)


MPLS L2VPN就是在MPLS网络上透明传递用户的二层数据。从用户的角度来看，这个MPLS网络就是一个二层的交换网络，通过这个网络，可以在不同站点之间建立二层的连接


# VPWS

## VPWS基本架构

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gi9phz8sg5j30zy0bgq98.jpg)


* 接入电路AC（Attachment Circuit）：一条连接CE和PE的独立的链路或电路。AC接口可以是物理接口或逻辑接口。
* 虚电路VC（Virtual Circuit）：两个PE节点之间的一种逻辑连接。
* 隧道Tunnel（MPLS Tunnel）：用于在PE之间透明地传输用户数据。

> VC提供用户二层数据穿越运营商骨干网络的通道，可以将其简单地理解为连接两个AC接口虚拟线路（点到点连接），将两条用户侧的AC“短接”起来。因此在MPLS L2VPN的实现中，VC又被称为PW（Pseudo Wire，伪线）。

## VPWS 报文发送过程

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gi9pmfnnvpj31ac08sdjo.jpg)

* MPLS L2VPN通过标签栈实现用户报文在MPLS网络中的透明传送
* 外层标签（称为Tunnel标签）用于将报文从一个PE传递到另一个PE。
* 内层标签（在MPLS L2VPN中称为VC标签）用于区分不同VPN中的不同连接，接收方PE根据VC标签决定将报文转发给哪个CE（哪个AC接口）。

> 华为查询方式：
>
> display mpls l2vc interface XXX ：能查询内层VC标签，查询到VC tunnel下的tunnel id
>
>display tunnel-info tunnel-id X ： 能查询出接口、外层标签



## VPWS实现方式

### Martini
Martini是基于LDP的，Martini方式使用两层标签，内层标签是采用扩展的LDP作为信令进行交互。

PE之间建立LDP的remote session，PE为CE之间的每条连接分配一个VC标签。二层VPN信息将携带着VC标签，通过LDP建立的LSP转发到remote session的对端PE。

在Martini方式中，两个CE间的VC Type + VC ID来识别一个VC。同一个VC Type的所有VC中，其VC ID必须在整个MPLS网络中唯一。
* VC-Type：表明VC的封装类型，例如ATM、VLAN或PPP。
* VC-ID：标识VC。相同VC-Type的所有VC，其VC-ID必须在整个MPLS网络中唯一。


### Martini的标签协商-VLL技术

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gia20woay1j30wt07xdgk.jpg)


在PE1上配置：

首先在PE之间配置 mpls ldp remote 会话
```
mpls ldp remote-peer PE2
    remote ip 3.0.0.42
```
之后在PE1接入CE的接口上创建L2VPN连接
```
mpls l2vc 3.0.0.42 1001
```

同样的，在PE2上配置：
```
mpls ldp remote-peer PE1
    remote ip 3.0.0.44

mpls l2vc 3.0.0.45 1001
```


这样，PE1和PE2之间就建立了一条VC（也就是PW），VC_ID为1001

之后，先在PE1上的GE子接口上绑定L2VC，之后PE1会为该VC分配一个内层标签140288，放入mapping消息中，并发送request和mapping消息给PE2，由于PE2并没有配置L2VC，所以将不做回应

然后，在PE2上的GE子接口上配置L2VC，此时PE2会为该VC分配一个标签140290，将这个标签放在mapping消息中，并发送mapping和request报文给PE1。PE1接收到PE2发送过来的request和mapping消息后，作出反应，将自己分配的标签140288放 入mapping消息中，并发送mapping消息给PE2，再检查PE2发过来的mapping消息发现VC信息符合，便将mapping里面的标签140290放入本地，作为PE1的Remote Label，并使本地的VC状态up，如下图示。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gia44kmgwrj30x109g3za.jpg)


PE2接收到PE1端过来的mapping报文，比较之后发现VC信息相符，则将mapping中的标签140288放入自己本地，作为PE2的Remote Label，将VC的状态改为UP，此时VC建立完毕，如上图示。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gia45j344ej30vs08vwf9.jpg)


### Martini的转发过程

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gia46vkjmej30wg0bvq3u.jpg)


报文从CE1到达PE1，根据接口转发表查得公网标签（1027）和VC标签(140290)，打上标签从出接口转发出去。倒数第二跳的时候弹出标签，在PE2上查找VC标签对应的出接口，弹出私网标签，转发到CE2。


### 一些命令

```
Display mpls ldp session

Display mpls ldp peer

Display mpls l2vc brief

Display mpls l2vc interface xxxxxxx

display tunnel-info tunnel-id xxx

display mpls l2vc vc-id
```



### Kompella

Kompella是另一种VLL技术的主流模式，与MPLS L3VPN很相似，特别是CE、PE、ROUTE-TARGET、RD、SITE的定义以及用途

区别是kompella传送的是二层信息，而MPLS BGP VPN传送的是三层路由信息，为此kompella进行了相应的BGP NLRI扩展


Kompella方式采取标签块的方式，一次为多个连接分配标签。用户可以指定一个本地CE的范围（CE range），表明这个CE能与多少个CE建立连接。系统会一次为这个CE分配一个标签块，标签块的大小等于CE range。这种方式允许用户为VPN分配一些额外的标签，留待以后使用。这样会造成标签资源的浪费，但是同时带来一个很大的好处：减少VPN部署和扩容时的配置工作量。


#### 标签块
* Kompella方式采用标签块（Label Block）分配标签，一次为多个连接分配标签。

标签块的三要素：
* 标签块的起始标签LB（Label Base）：由系统分配，表明该标签块的起始标签。
* 标签块的大小LR（Label Range）：用户可以指定一个本地CE的范围（CE range），表明这个CE能与多少个CE建立连接。系统一次为CE分配一个大小等于CE range的标签块。
* 偏移量LO（Label-block Offset）：标签不够用时，为了不破坏原有的VC连接，需要给CE分配一个新的标签块。多个标签之间的关系通过LO来定义，标志前面所有已分配的标签块大小的总数。


#### 配置VC连接



![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gilo6nv22oj30rt07aq5v.jpg)

进入MPLS-L2VPN视图：
```
mpls l2vpn l2vpn-name
```
创建CE，并进入MPLS-L2VPN-CE视图
```
ce ce-name id ce-id [range ce-range] [default-offset ce-offset ]
# ce-name ： 在当前PE的当前VPN上指定CE名称
# id ce-id : VPN内CE的ID值 



```
为CE创建连接
```
connection [ ce-offset id ] interface interface-type interface-number [ tunnel-policy policy-name ] [ raw | tagged ]


# ce-offset id：与L2VPN相连的对端CE的ID。
# interface interface-type interface-number ： 与CE相连的接口，其封装格式必须与所属VPN一致。


```


#### Kompella

从图中可以看到，CE1与 CE3和CE13 都是点到点的连接，PE1会分配两个标签块，一个是用来与CE1建立连接的，一个是用来与CE13建立连接的，同样，PE2也给CE3分配了标签块，PE3给CE13分配了标签块
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gimg08533gj30z40iuwoh.jpg)












# VPLS
## VPLS基本结构

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gi9q1b3vzmj314e0j7k38.jpg)

* PW（Pseudo-Wire）：伪线。PW是VPLS中两个PE间的虚拟逻辑连接，在两个PE之间传输用户数据。
* VSI（Virtual Switch Instance）：虚拟交换实例。每个VSI提供单独的VPLS服务。通过VSI，可以将VPLS的实际接入链路映射到各条虚链接上。
* AC（Attachment Circuit）：接入线路。指CE与PE的连接，在L2VPN中，CE通过AC接入到PE。
* Tunnel（Network Tunnel）：隧道。用于承载PW，一条隧道上可以承载多条PW，一般情况下为MPLS隧道。隧道是一条本地PE与对端PE之间的直连通道，完成PE之间的数据透传。



## VPLS基本工作原理

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gi9sctbxn7j312p0fyjxu.jpg)

可以将每个VSI看作一台独立的交换机。PE连 接CE的端口（AC接口）作为交换机的端口，而连接各个PE的PW就是交换机的内部交叉电路。

## VPLS与普通L2VPN的区别

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gi9sfozbwpj30yq0jzwmz.jpg)

## VPLS基本拓扑

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gi9sk3h7khj30tj0d8n48.jpg)

同一个VSI中各个PE间必须建立**全连接**的PW，同时配置“水平分割”策略（无需配置，产品缺省），即如果PE收到远端PE发来的广播流量，它只向同一VPLS的所有AC端口转发流量，不向其他PE转发 。PE间全连接和水平分割一起保证了VPLS转发的可达性和无环路。



## VPLS控制平面

华为产品支持使用LDP或BGP实现VPLS的控制平面的功能，分别称为Martini方式的VPLS和Kompella方式的VPLS。

### Martini方式的VPLS
采用LDP作为信令，需要手工指定PE的各对等体，由于同一VPLS中各PE之间需要建立全连接，每当有新的PE加入时，所有相关PE上都修改配置，导致可扩展性较差。但是PW实际是点到点链路，使用LDP进行PW的建立、维护和拆除更为有效。

#### 配置

```
# 创建一个名称为company1的Martini方式VPLS的VSI，配置当前VSI实例的ID为1。
<HUAWEI> system-view
[HUAWEI] mpls lsr-id 1.1.1.1 
[HUAWEI] mpls
[HUAWEI-mpls] quit
[HUAWEI] mpls l2vpn
[HUAWEI-l2vpn] quit
[HUAWEI] vsi company1 static
[HUAWEI-vsi-company1] pwsignal ldp
[HUAWEI-vsi-company1-ldp] vsi-id 1


# 配置VSI对等体，建立与一个或多个对端PE的PW连接
# peer peer-address [ negotiation-vc-id vc-id ] pw pw-name
# negotiation-vc-id vc-id ： 虚电路的唯一标识，一般用于两端VSI ID不同但要求互通的情况，参数vc-id不能与本端其他VSI配置的VSI ID相同，到同一个Peer的negotiate-vc-id指定的VC ID也不能相同
# pw pw-name ： PW的名称，用于标识该PW，PW名称要求在同一VSI下唯一，在不同VSI下PW名称可以相同。

<HUAWEI> system-view
[HUAWEI] vsi aa static
[HUAWEI-vsi-aa] pwsignal ldp
[HUAWEI-vsi-aa-ldp] vsi-id 1
[HUAWEI-vsi-aa-ldp] peer 1.1.1.1
[HUAWEI-vsi-aa-ldp] peer 1.1.1.1 pw pw1



# l2 binding命令用于将二层接口绑定到VSI实例
# l2 binding vsi vsi-name [ access-port ]

```

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gimhjbge83j30wi0jfjxa.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gimhjqd5jcj30g90it0u8.jpg)


### Kompella方式的VPLS

采用BGP作为信令，可以通过配置VPN Target实现VPLS成员的自动发现，增加PE或删除PE时，所需的额外操作很少，因而具有较好的扩展性。


#### 配置


```
# 创建一个名称为company2的Komeplla方式VPLS的VSI

[HUAWEI] vsi company2 auto
[HUAWEI-vsi-company2] pwsignal bgp

route-distinguisher route-distinguisher 配置VSI的RD
vpn-target vpn-target&<1-16> [ both | export-extcommunity | import-extcommunity ] 配置VSI的VPN-Target

# 配置Site连接
site site-id [ range site-range ] [ default-offset { 0 | 1 } ] ]

# 配置接口绑定VSI
l2 binding vsi vsi-name
```

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gimhw2iexjj30wo0h70vv.jpg)