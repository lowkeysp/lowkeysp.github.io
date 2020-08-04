---
title: MPLSVPN-红茶
date: 2020-07-09 20:29:24
tags: ['MPLS','VPN']
categories: 数据
---






# MPLS VPN
优势：
* PE路由器与CE路由器运行动态路由协议
* PE为每个客户维护一个独立的路由表
* 允许客户使用重叠的地址空间

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggxpof8g7xj30i40byjxg.jpg)





# MPLS 初体验

## 路由层面

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggybkm1rc2j310u0eo7k9.jpg)

* PE会创建两个VRF，用于区分两个不同的客户，同时用于解决从客户上收过来的路由冲突
* 两个PE之间建立MP-BGP，用于传递vpnv4的路由表（vpnv4路由：RD值：ip地址/掩码，比如：4813：12：10.1.1.0/24）。RD值用来防止地址冲突。
* 另一端的PE路由器使用RT值来区分这两个路由，RT值可实际上是扩展的community，默认另一端的PE路由器的VRF都是拒收任何路由的，配上import的RT值之后，则可以接受该RT值的vpnv4的路由。然后将接受的vpnv4路由再转给相应的客户。



可以看出，RD值和RT值是用来转发路由的，是路由层面的，和数据转发层面没有关系。

## 数据层面

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggybp11u3jj311q0f2dsf.jpg)

* 100的标签是左边的PE设备给右边通告的，用于区分左边的不同客户，当一个带有100标签的数据包传给左边的PE设备后，左边的PE设备就知道这个数据应该是给上面的黄色客户的。这个标签值也是通过MP-BGP协议传递给右边的PE设备的，和RT值传递的方式一样。这是内层的标签，可以说，内层的标签用来区分客户
* 301的标签是通过MPLS协议，P设备给右边的PE设备的，也就是建立双向的LDP。外层的标签用于在骨干网的转发。

可以看出，内层标签是通过MP-BGP传递，外层标签则是MPLS，通过两层标签来进行数据层面的转发。

顶层标签有PHP机制，但是VPN标签不能再倒数第二跳弹出，因为Exgress PE需要这层标签


外层标签的数据转发是这样的，当PE2给PE1传递MP-BGP路由时，next-hop会是自身，然后，客户给PE1传递数据包后，PE1查找下一跳为PE2，然后发现使用的是MPLS，则会加上外层标签进行传递，而不是基于IP地址进行传递。

# MPLS VPN架构 -PE router

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggybxxatlaj310m0ldtq9.jpg)

一个PE设备会维护一个全局路由表和多个VRF路由表。





## RD值

用来区分冲突的私网路由，64bit

* 用于在MP-BGP运载VRF的IPv4路由前缀时，确保这些前缀的唯一性
* RD并不会说明该前缀属于哪一个VRF（需要搭配RT）
* 64bits的RD与32bit的IPv4前缀构成96bit的VPNv4前缀
* 如果不同的VPN客户，存在相同的IPv4地址空间，那么可以通过设置不同的RD值从而保障前缀的唯一性


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggycqgnotzj310j0juqit.jpg)


# RT值
* RT值实际上就是MP-BGP的扩展的community
* 用于区分VPN（Customer）
* 一条路由可以附加多个RT值
* import和export

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggycse0c3lj311l0gl12m.jpg)




# VRF MODEL

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggkzk3a036j311l0k4k29.jpg)


* 一个VRF需要定义RD值和RT值（export和import）。默认不配import的话，是拒收任何路由的。
* 一个VRF可以有多个接口，一个接口只能属于一个VRF


一个VRF可以想象成是一个虚拟路由器，有路由表，转发表，属于该VRF的接口，还有与VRF关联的其他信息，如RD值和RT值

## MP-BGP
* BGP 能够承载大批量的路由前缀
* BGP 拥有丰富的路径属性及丰富的策略部署工具
* BGP 能够在非直连的Peer之间交互路由信息，因此P路由器无需运行BGP即无需维护客户的路由信息，大大减轻了负担

MP-BGP在MPLS VPN中用于在PE路由器之间交互VPNv4路由， 同时为VPNv4路由分配标签

IPv4路由重发布进MP-BGP（如果PE-CE之间跑的就是BGP，则不用重发布路由）

一个MP-BGP的update报文所包含的元素有：
* VPNv4地址
* 扩展的community属性，如RT值，SOO等
* 用于VPN报文转发的标签
* 其他的普通的BGP的属性，如MED等等

# 路由汇总对MPLS VPN数据传输的影响

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggydpep9qnj311q0n8aop.jpg)

如果在P路由器上对PE的loopback地址做了汇总，由于汇总路由是在这台P设备上产生的，因此PHP机制在这个P设备上起效，在P设备上就进行了标签弹出，由于这台P设备面对的是一个VPN标签，但是这个P设备不认识这个VPN标签，那么就会丢弃。




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

 
# 配置完之后，这个接口就不属于全局路由表中了，不会在全局路由表中出现这个接口，而是在VRF的虚拟路由表中，可以通过
show ip route vrf cisco 
中查看vrf的虚拟路由表

```
* PE配置MP-MGP
```
[config] router bgp 234
[config-router] bgp router-id 2.2.2.2
[config-router] no bgp default ipv4-unicast # 如果我们不需要PE1与PE2之间交互IPv4前缀的话，可以将默认建立Ipv4的BGP邻居开关关掉
[config-route] neighbor 4.4.4.4 remote 234
[config-router] neighbor 4.4.4.4 update-source loopback0
[config-router] address-family vpnv4
[config-router-af] neighbor 4.4.4.4 activate
[config-router] neighbor 4.4.4.4 send-community extended



#查看MP-BGP的邻居
[cisco] show ip bgp vpnv4 all summary
```


* 完成PE-CE之间路由的重发布
```
[config] router bgp 234
[config-router] address-family ipv4 vrf cisco #必须在这个cisco中进行重发布
[config-router-af] redistribute ospf 1 vrf cisco match internal external 
# internal表示只重发布ospf的内部路由，external表示只重发布ospf的外部路由
 

PE给CE的重发布
[config] router ospf 1 vrf cisco
[config-router] redistribute bgp 234 subnets

```

* 查看vpnv4的路由
```
show ip bgp vpnv4 all

show ip bgp vpnv4 all X.X.X.X #查看某一条vpnv4路由
```
* 查看本地给路由分配的标签,也就是内部标签
```
show ip bgp vpnv4 all labels
```

* 查看vrf路由表
```
show ip route vrf cisco

```
* 查看本地配置的vrf以及其下关联的接口
```
show ip vrf
```
* 查看vrf详细信息
```
show ip vrf detail
```
* 查看vrf接口
```
show ip vrf interface
```
* 查看VRF的CEF表
```
show ip cef vrf
```
* 查看VRF CEF表的详细信息
```
show ip cef vrf detail
```
* ping
```
ping vrf vrfname dst-addr
```
* traceroute
```
traceroute vrf vrfname ? 
```





# RR 
* 在MPLS VPN网络中，RR和其他BGP设备（PE）的工作方式有所不同，在RT没有在RR配置的时候，RR并不会拒绝VPNv4路由，这点和PE不一样，PE路由器如果收到一条没有在任何RT输入到VRF的VPNv4路由的话，该路由就会被拒绝，PE采用这种方式来节省内存

* 没有必要让一个RR或一组RR拥有或反射BGP表里所有的VPNv4路由。可以将这些VPNv4路由分成几组，然后让多个RR或多组RR分别承载这几组路由。操作的方式是，在VPNv4地址簇模式中通过命令bgp rr-group extcommunity-list来实现，这个扩展的community-list用来指定希望通过这个RR允许或拒绝RT。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggykexytyyj312j0kwqfv.jpg) 

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggykgbgn60j30z60nuguv.jpg)





# PE与CE之间的路由协议

## EBGP
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggyl4g5ntij31180nr4cj.jpg)

那么PE从CE学到的路由，直接就是BGP的路由条目，这个VRF-A routing table就是BGP路由表，那么，只需要加上RD前缀，RT值，标签等就能直接变成vpnv4路由传给对端PE，但是如果PE与CE是OSPF连接的话，那么学到的是OSPF路由条目，VRF-A routing table也是OSPF路由表，那么就需要重发布到MP-BGP中。

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggyljnpanxj311v0o4k6s.jpg)


## 静态路由

在PE上做配置
```
# 需要在vrf中做
ip route vrf cisco 1.1.1.1 255.255.255.255 10.1.12.1 e0/0

# 重发布
router bgp 234
  address-family ipv4 vrf cisco
    redistribute static

```

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggyl6uj7zuj31140ogn96.jpg)

# SOO
使用SOO用于防环路,Site of Origin

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggym74mybcj311b0foak6.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggymaqh4tnj310j0hu4d5.jpg)


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggymbd64imj310u0fxtnd.jpg)


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggymc688ubj310u0hcwtk.jpg)


解决方法：

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ggymfglmsdj311e0jltru.jpg)