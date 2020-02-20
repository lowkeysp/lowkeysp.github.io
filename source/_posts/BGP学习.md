---
title: BGP学习
date: 2020-02-10 22:07:37
tags: ['网络','BGP']
---


# 自制系统（Autonomous System，AS）
AS号范围：1-65535，其中，64512-65535属于私有AS号

# BGP相关特性
* BGP报文是基于TCP端口号179。
* 是单播建立邻居（手工指邻居）。没有组播和广播
> 注：由于BGP报文是基于TCP端口号179的，也就是BGP协议是一个三层之上的协议，说明BGP的邻居建立是不需要直连的，两个BGP对等体之间只要能TCP协议可达，且179端口号没有被占用的话，则就可以建立邻居. 一个运行了BGP协议的路由器也就是一直在监听这个179端口号
* 支持VLSM,CIDR
* 以AS为单位的路径矢量属性的路由协议
* 会话开始的时候会发送整张路由表,以后会触发更新[触发式增量更新]
* 有丰富的路径属性,简单的路由算法

# BGP基本配置及邻居建立
BGP有两种类型的邻居关系: EBGP(不同AS之间的邻居关系)和IBGP(一个AS内的邻居关系)


## EBGP

假设有两台路由器分别属于两个AS, 路由器1在AS-100中,路由器2在AS-101中, 这两个路由器相连的端口地址分别是12.1.1.1(路由器1)和 12.1.1.2(路由器2)(12.1.1.0/24)

路由器1配置
```

router bgp 100 #  router bgp AS号 指定这个路由器所在的AS号

bgp router-id id号   # 指定router-id,通常是loopback地址

neighbor 12.1.1.2 remote-as 101 # neighbor 对端地址 remote-as AS号   指定邻居, 还得标注对等体所在的AS号,从而能知道是使用EBGP还是IBGP

```
路由器2使用同样的配置

检验BGP是否已经起来,使用
```
show ip bgp summary

# 打印出来后,有一个TblVer列,这列说明当路由表发生变化时,则版本会加1
```
还可以使用
```
show tcp brief #查看179端口 TCP是否已经建立
```


如果使用loopback地址建立邻居的时候,需要特别注意的是,要设置**多跳特性**.因为默认情况下,EBGP的跳数为1,(而IBGP的默认跳数是255跳)因此,需要设置跳数大于1.比如设置成2,如下:

```
neighbor 对端loopback地址 ebgp-muiltihop 2
```
> 通常，设置成2，而且不推荐使用比2大的数。因为GTSM(通用TTL安全机制)，如果设置成2的话，则肯定是对端**直连**路由器的loopback地址，而如果设置成比较大的数，比如255，则有可能黑客会冒充路由器和你连接，黑客假如在250跳这里等着你，则这样就不安全，而如果设置成2的话，则就能保证只能直连的才能建立连接。

而且如果使用loopback地址建立邻居的话,还需要更新源，

```
neighbor 新的地址  remote-as AS号
neighbor 新的地址 update-source 端口标识(如 loopback 1) #更新源，如果不更新源的话，则会使用端口地址和对方的loopback地址去建立tcp连接，这显然是不对的，而且也是建立不成功的。所以这条命令就是说我用我的loopback地址去和对方建立邻居关系
```



## IBGP

IBGP建立不需要直连,只要两端的TCP 179端口 通就可以

假设在AS-101中有两台路由器,一台是路由器2(上面的路由器2),一台是路由器3,一台是路由器5.其中,路由器2和路由器5没有直连,而是通过路由器3连接.

路由器2和路由器3直连(23.1.1.0/24):
> 路由器2的地址:23.1.1.2 
>
> 路由器3的地址:23.1.1.3

路由器3和路由器5直连(35.1.1.0/24):
> 路由器3的地址:35.1.1.3
>
> 路由器5的地址:35.1.1.5


因此,如果路由器2和路由器5之间要建立IBGP的话,前提条件是要保障路由器2和路由器5是三层可达的.可以使用动态路由或者静态路由保证可达.

如果是静态路由的话,在路由器2上配置一条静态路由
```
ip route 35.1.1.0 255.255.255.0 23.1.1.3
```
路由器5上配置静态路由
```
ip route 23.1.1.0 255.255.255.0 35.1.1.3
```

三层可达之后,就可以使用上面EBGP的命令去建立邻居关系


更新邻居建立的端口地址可以使用
```
neighbor 新的地址  remote-as AS号
neighbor 新的地址 update-source 端口标识(如 loopback 1) 
# 两个命令一起使用,先指定对端的as号,然后再两个对等体地址更新
```
## 问题1:
> 问题:RIP协议是使用的是UDP的520端口,而BGP协议使用的是TCP的172端口,那么,如果使用RIP协议的单播模式,能否可以实现非直连,像IBGP一样
>
>答:不可以.因为虽然RIP也得需要三层可达,使用UDP协议.但是 RIP的跳数只能是1跳.因此,只能直连,而IBGP的默认跳数是255跳.EBGP的默认跳数是1跳.但是,BGP的跳数是可以修改的,但是RIP的跳数是不能修改的.因此,RIP协议只能直连



# B站视频的BGP学习

链接：https://www.bilibili.com/video/av49497733?p=1


该教程使用的是华为的路由器
## BGP的EBGP邻居

构建BGP邻居的命令在上面已经给出了，这里不再重复

当你使用`display bgp peer`查看邻居关系时，解释
```
[Huawei]display bgp peer

 BGP local router ID : 115.168.1.1
 Local AS number : 65530
 Total number of peers : 2		  Peers in established state : 2

  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State PrefRcv

  59.43.1.1       4        4809       13       13     0 00:11:35 Established     0
  115.168.1.2     4       65530       13       14     0 00:11:35 Established     0


```
  其中，
  * MsgRcvd表示接受的报文
  * MsgSent表示发送的报文
  * OutQ表示出去的队列，如果链路很拥塞的话，则这个队列就不是0了
  * PrefRcv表示接受到BGP peer的路由条目数

  还可以通过`display bgp peer verbose`查看更详细的信息

  在两个AS之间是不能运行IGP的，因此需要**静态路由**


## BGP的IBGP邻居和路由

* 传输协议：TCP，端口号是179
* 无需周期性更新
* 路由更新：只发送增量路由
* 周期性发送KeepAlive 报文检测TCP的连通性

Loopback地址接口无法关闭，只要设备开启，则loopback接口就会开启

BGP路由产生的方式

1）network 方式

需要在BGP进程下进行配置
```
bgp AS号
network IP地址 子网掩码
```
该命令表示该设备上如果存在该IP地址的路由（通过IGP或者直连路由产生），则使用该命令可以产生该IP地址的BGP路由

通过这种方式，就进入到了BGP路由表中。可以通过`display bgp routing-table`查看。（注：BGP Routing-Table只是BGP的路由表，并不代表最佳路由，比如去一个目的地会有多个路径，它会都放在这个BGP路由表中，真正的BGP最佳路由可以使用 `display ip routing-table protocol bgp`查看）。

使用这种方式引入路由的起源属性是IGP

2）引入路由（import-route）方式

通过`import-route`命令可以把其他协议学习到的路由引入BGP路由中。
`import-route`后面可以是不同的协议，如下
```
  direct  Connected routes
  isis    Intermediate System to Intermediate System (IS-IS) routes
  ospf    Open Shortest Path First (OSPF) routes
  rip     Routing Information Protocol (RIP) routes
  static  Static routes
  unr     User network routes
```
使用这种方式产生路由的起源属性是incomplete。

3）聚合路由（汇总路由）

使用这种方式引入路由的起源属性是IGP


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gby3ofyl6xj30ih0bq40z.jpg)

* Adj-RIB-In :对等体宣告给本地speaker的未处理的路由信息库
* Adj-RIB-Out ：本地Speaker宣告给指定对等体的路由信息库

从图中可以看出，从对等体传来路由信息后，首先会经过一个输入策略引擎，进行一个过滤，也就是需要哪些，丢弃哪些，然后过滤后的会进入到BGP路由表中（Loc-RIB），BGP路由表会将最佳路由放入到IP路由表中，还会把BGP路由传给输出策略引擎，进行过滤，需要将哪些路由传出去，哪些不传，然后传给对等体。


## BGP报文类型

(复制之前的BGP协议博客)

BGP的运行是通过消息驱动的，共有Open、Update、Notification、Keepalive和Route-Refresh等5种消息类型。

* Open消息：是TCP连接建立后发送的第一个消息，用于建立BGP对等体之间的连接关系。对等体在接收到Open消息并协商成功后，将发送Keepalive消息确认并保持连接的有效性。确认后，对等体间可以进行Update、Notification、Keepalive和Route-Refresh消息的交换。
* Update消息：用于在对等体之间交换路由信息。Update消息可以发布多条属性相同的可达路由信息，也可以撤销多条不可达路由信息。
> * 一条Update消息可以发布多条具有相同路由属性的可达路由，这些路由可共享一组路由属性。所有包含在一个给定的Update消息里的路由属性适用于该Update消息中的NLRI（Network Layer Reachability Information）字段里的所有目的地（用IP前缀表示）
> * 一条Update消息可以撤销多条不可达路由。每一个路由通过目的地（用IP前缀表示），清楚的定义了BGP Speaker之间先前通告过的路由。
> * 一条Update消息可以只用于撤销路由，这样就不需要包括路径属性或者NLRI。相反，也可以只用于通告可达路由，就不需要携带撤销路由信息了。

* Notification消息：当BGP检测到错误状态时，就向对等体发出Notification消息，之后BGP连接会立即中断
* Keepalive消息：BGP会周期性的向对等体发出Keepalive消息，用来保持连接的有效性。
* Route-Refresh消息：Route-Refresh消息用来通知对等体自己支持路由刷新能力（Route-Refresh capability）。
```
refresh  bgp all import / export

# export Trigger outbound soft reconfiguration 该参数是将自己的路由策略发送给对等体，会发出update报文
# import Trigger inbound soft reconfiguration 该参数是向对等体发出去route-Refresh报文，请求对等体将路由策略发给自己
```

在所有BGP交换机使能Route-Refresh能力的情况下，如果BGP的入口路由策略发生了变化，本地BGP交换机会向对等体发布Route-Refresh消息，收到此消息的对等体会将其路由信息重新发给本地BGP交换机。这样，可以在不中断BGP连接的情况下，对BGP路由表进行动态刷新，并应用新的路由策略。

### 状态机
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gbzhy4yprgj30h80b3ad9.jpg)

IDLE状态的原因：
1. 没有建立TCP连接。
2. EBGP通过loopback地址建立邻居时，没有修改EBGP-MAX-HOP大于1
3. 没有到达对端的路由
4. 在地址族中没有激活邻居，举例，
```
ipv4-family unicast
  undo synchronization
  undo peer 22.1.1.1 enable
```
上述说明了对端邻居22.1.1.1没有激活。默认情况下，只有IPv4单播协议默认激活邻居，其他协议默认不激活邻居（如Ipv6）

5. 管理性的关闭了邻居 Idle（Admin）。如使用`peer 22.1.1.1 ignore`关闭邻居


## BGP路由通告原则

建立连接时，BGP Speaker 只把本身用的最优路由通告给对等体
（可以通过命令`dis bgp routing-table peer 22.1.1.1 advertised-routes`查看，表示向22.1.1.1通告的路由条目有哪些）

多条路径时，BGP Speaker 只选最优的路由放入路由表

BGP Speaker 从EBGP获得的路由会向它所有的BGP对等体通告（包括EBGP和IBGP，除了给这个路由的BGP对等体）

BGP Speaker 从IBGP获得的路由不会通告给它的IBGP邻居（防止环路，EBGP防环是通过AS号码。如果接受的路由中的AS-Path包含了和自身相同的AS，则拒绝接受该路由）

BGP Speaker 从IBGP获得的路由是否通告给它的EBGP对等体要依IGP和BGP同步的情况来决定。（BGP和IGP同步的概念：BGP Speaker 不将从IBGP对等体对等体获得的路由信息通告给它的EBGP对等体，除非该路由信息也能通过IGP获得）。如果没有同步的话，则不会有有效路由，这样也就不会更新给EBGP了，同步之后会有有效路由。（华为设备不支持同步）


## BGP路由下一跳


在BGP中，每经过一个AS叫做一跳。所以当一个router从一个EBGP邻居学来的路由宣告给IBGP邻居的时候,不会更改这个路由的下一跳地址，所以当IBGP邻居学到这个路由的时候，下一跳往往是AS外面的一个地址。对于这个router来说是不可达的。**它在BGP路由表中存在这一条路由，但是这条路有前面没有>，也就是没有最佳路由选入到IP路由表中**。对此，解决的办法是，将从EBGP学来路由的路由器在宣告这条路由给它的IBGP邻居的时候，宣告下一跳地址为自己，使用的命令为：
```
peer 3.3.3.3 next-hop-local
#这条命令表示将BGP的路由发送给3.3.3.3这个邻居时,将路由的下一跳设置成自己的地址，这个地址是与3.3.3.3建立邻居所使用的源地址
```

## BGP路由黑洞

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc0qn7pn3ij30qo0c8neo.jpg)

当我们在R3上没有运行BGP进程，我们在R5上关闭了同步，这时它会将一条并没有优化的路由传送给R7，当R7要发向R1发包时，它看到R5是它的下一跳，于是将包发给R5，然后R5又查看它的路由表，发现到R1的下一跳是R2，并继续查找，发现在通过R3可以达到R2，于是它将数据送给R3，这时问题出现了，因为R3没有运行BGP，它不知道R1怎么走，于是它将数据包丢弃，从而造成路由黑洞

由此可见，BGP与IGP同步的重要性。（华为设备不支持，Cisco设备支持）


BGP路由黑洞的解决方案：
### Full Mesh（全互联）
所有路由器运行BGP进程，且一个AS内所有路由器两两之间需要建立IBGP邻居关系

> 问题：过多的TCP会话以及没有必要的路由更新

### 路由反射器
会专门讲解路由反射器

## 对等体组

对等体组是为了简化重复的配置，因此，有了对等体组的概念
```
group 对等体组-名字 internal/external
# internal表示是IBGP对等体，externa表示是EBGP对等体

#然后，就能使用对等体组进行配置，和对等体的配置是一样的，只不过换成了对等体组名字
peer 对等体组-名字 connect-interface LoopBack0
...

#将44.1.1.1假如到这个对等体组中去
peer 44.1.1.1 group 对等体组-名字

```

## BGP路由聚合

路由聚合可以将具体路由聚合成一条聚合路由

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc1sf72sumj30h7093gp6.jpg)

### 路由聚合原则
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc1shgi2naj30ig07vmzq.jpg)

1)自动聚合

对BGP引入的子网络路由进行自然掩码聚合。配置自动聚合后，生成聚合后的自然网段路由，而原引入的子网路由被抑制，不会被优选和发布给BGP邻居
（也就是仅对引入路由有效）
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc1u08dry4j30jd075tcl.jpg)



2）手动聚合

IGP协议，直连路由等先进行聚合路由，然后再加入到BGP路由当中

3）aggregate

要求本路由器至少有一条明细路由才能进行手工聚合,并且要使用network通告明细路由

```
aggregate (聚合路由，Aggregate address) 掩码 参数
# 参数可以是
  as-set             Generate the route with AS-SET path-attribute
  attribute-policy   Set aggregation attributes
  detail-suppressed  Filter more detail route from updates
  origin-policy      Filter the originate routes of the aggregate
  suppress-policy    Filter more detail route from updates through a Routing
                     policy 
```

* Detail-suppressed : 对于华为路由器来讲，在进行BGP的手工聚合时，默认不会抑制明细路由，即在手工聚合以后，会将汇总路由和明细路由都通告给邻居。使用Detail-suppressed可以抑制明细路由的通告，只通告汇总路由给邻居
* AS_set ：携带路由起源的AS属性，用于防环。当在非明细路由起源的AS聚合路由时，默认会丢弃明细路由的起源AS属性，这样可能导致环路，可添加AS_set属性则会携带明细起源的AS号到路径属性的最右边，如果明细从多个AS产生，明会使用｛｝括起来。
* Attribute-policy ：可以更改路由的属性，比如一条路由的起源属性更改为incomplete.
* Suppress-policy : 抑制策略，如果使用Detail-suppressed则会抑制所有的路由，而使用Suppress-policy则只会抑制被route-policy匹配的路由。

下面通过实验来说明

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc232gvn4nj30s80jsq4w.jpg)

R4和R2 R3建立IBGP邻居，R1同R2和R3建立直连的EBGP邻居，R1上有三个环回口11.1.1.1/32，11.1.1.2/32，11.1.1.3/32，将它们通告到BGP中

* 自动聚合

在R1上引入直连路由，并进行自动聚合
```
R1：
bgp 100
summary automatic #自动聚合只对重分布的路由生效，对于network的不生效。
import-route direct
```
在R2上查看结果，发现已经能看到这条汇总路由了。
```
R2：
<R2>display bgp routing-table

> 11.0.0.0 123.1.1.1 0 100? 已经没有掩码了
> 123.0.0.0 123.1.1.1 0 100?
```

* aggregate方式

在默认情况下，手工聚合不抑制明细路由，明细和汇总路由会一起通告给邻居
```
[R1-bgp]display this
[V200R003C00]
#
bgp 100
router-id 11.1.1.1

aggregate 11.1.1.0 255.255.255.252
network 11.1.1.1 255.255.255.255
network 11.1.1.2 255.255.255.255
network 11.1.1.3 255.255.255.255
```
在R1上查看BGP表，发现有明细路由和汇总路由，都是有效且最优的
```
[R1-bgp]display bgp routing-table
> 11.1.1.0/30 127.0.0.1 0 i
> 11.1.1.1/32 0.0.0.0 0 0 i
> 11.1.1.2/32 0.0.0.0 0 0 i
> 11.1.1.3/32 0.0.0.0 0 0 i
```

在R2上查看BGP表，可以看到明细，也可以看到汇总路由
```
<R2>display bgp routing-table
> 11.1.1.0/30 123.1.1.1 0 100i
> 11.1.1.1/32 123.1.1.1 0 0 100i
> 11.1.1.2/32 123.1.1.1 0 0 100i
> 11.1.1.3/32 123.1.1.1 0 0 100i
```

* 在R1上通告汇总路由并且抑制明细
```
R1：
bgp 100
aggregate 11.1.1.0 255.255.255.252 detail-suppressed
network 11.1.1.1 255.255.255.255
network 11.1.1.2 255.255.255.255
network 11.1.1.3 255.255.255.255
```

R1上查看BGP表
```
[R1-bgp]display bgp routing-table # 明细路由被抑制了，不会通告给邻居
*> 11.1.1.0/30 127.0.0.1 0 i
s> 11.1.1.1/32 0.0.0.0 0 0 i
s> 11.1.1.2/32 0.0.0.0 0 0 i
s> 11.1.1.3/32 0.0.0.0 0 0 i
```
可以看到，明细路由已经不是最佳路由了 ，被抑制了，所以是通过让这些明细路由不是最佳路由，这样就不会向邻居通告了

```
<R2>display bgp routing-table #在R2上已经看不到明细路由了
*> 11.1.1.0/30 123.1.1.1 0 100i
```

* 在R2上聚合11.1.1.0/30的路由，添加AS_set属性

我们取消在R1上做的手工聚合，改成在R2上做聚合，如果不加AS_set，则这条聚合路由在AS200中默认不是没有AS_path属性的。

我们在R2上进行聚合，则如果不加AS_SET的话，则不会将AS-100添加到AS_path中的，虽然明细路由是从AS-100中来的，其实都没有AS_Path属性

```
R2：
bgp 200
aggregate 11.1.1.0 255.255.255.252
```

在R2和R4上查看BGP表，这条路由都没有AS_path属性。

```
<R2>display bgp routing-table

*> 11.1.1.0/30 127.0.0.1 0 i
```

```
<R4>display bgp routing-table

*>i 11.1.1.0/30 22.1.1.1 100 0 i
```

在R2上将聚合路由带上as_set属性，这样会在聚合路由上添加明细路由起源自哪个AS，用于防环(如何防环？在R2进行路由聚合，带上AS-100之后，则R2就不会将这条路由再通告给R1了，如果通告给R1，那就形成了环路)。如果这些明细来自不同的AS，则会在AS_path的最右边使用｛｝将产生明细的多个AS放入｛｝中。

```
[R2-bgp]aggregate 11.1.1.0 255.255.255.252 as-set
```
接着在R2和R4上查看BGP表

```
[R2-bgp]display bgp routing-table

*> 11.1.1.0/30 127.0.0.1 0 100i
```
```
<R4>display bgp routing-table
*>i 11.1.1.0/30 22.1.1.1 100 0 100i
```

* 更改聚合路由的属性，将起源属性更改为未完成

```
R2：

#路由规则
route-policy ATT1 permit node 10
apply origin incomplete #改成incomplete

aggregate 11.1.1.0 255.255.255.252 as-set attribute-policy ATT1
```

查看R2的BGP表，发现11.1.1.0/30这条汇聚路由的起源属性变成了?
```
<R2>display bgp routing-table
*> 11.1.1.0/30 127.0.0.1 0 100?
```
```
<R4>display bgp routing-table
*>i 11.1.1.0/30 22.1.1.1 100 0 100?
```

* 在R2上聚合11.1.1.0/30，只抑制11.1.1.2这一条明细路由

```
[R2-bgp]aggregate 11.1.1.0 255.255.255.252 as-set attribute-policy ATT1 suppress-policy supp

route-policy supp permit node 10
if-match acl 2000

acl number 2000
rule 10 permit source 11.1.1.2 0
```

查看R2的BGP表，发现11.1.1.2/32的路由是存在的（被抑制），但这条路由在R4上是不存在的。
```
[R2]display bgp routing-table

> 11.1.1.0/30 127.0.0.1 0 100?
> 11.1.1.1/32 123.1.1.1 0 0 100i
s> 11.1.1.2/32 123.1.1.1 0 0 100i
*> 11.1.1.3/32 123.1.1.1 0 0 100i
```

```
[R2]display bgp routing-table 11.1.1.2 32，被抑制的路由，没有通告给任何的邻居。

BGP local router ID : 22.1.1.1
Local AS number : 200
Paths: 1 available, 1 best, 1 select
BGP routing table entry information of 11.1.1.2/32:
From: 123.1.1.1 (11.1.1.1)
Route Duration: 00h08m12s
Direct Out-interface: GigabitEthernet0/0/2
Original nexthop: 123.1.1.1
Qos information : 0x0
AS-path 100, origin igp, MED 0, suppressed, pref-val 0, valid, external, best, select, active, pre 255
Not advertised to any peer yet
```
```
[R2]display bgp routing-table 11.1.1.1 32 像这条路由，就通告到R4上去了。

BGP local router ID : 22.1.1.1
Local AS number : 200
Paths: 1 available, 1 best, 1 select
BGP routing table entry information of 11.1.1.1/32:
From: 123.1.1.1 (11.1.1.1)
Route Duration: 00h08m38s
Direct Out-interface: GigabitEthernet0/0/2
Original nexthop: 123.1.1.1
Qos information : 0x0
AS-path 100, origin igp, MED 0, pref-val 0, valid, external, best, select, active, pre 255
Advertised to such 1 peers:
44.1.1.1
```

```
<R4>display bgp routing-table

*>i 11.1.1.0/30 22.1.1.1 100 0 100?
i 11.1.1.1/32 123.1.1.1 0 100 0 100i
i 11.1.1.3/32 123.1.1.1 0 100 0 100i
```

## RR路由反射器 
参考：https://www.bilibili.com/video/av35766062?from=search&seid=13651713699760369094


由于IBGP的水平分割（从IBGP邻居学来的路由不会转发给其他的IBGP邻居），导致在AS内需要full-mesh（全互联），这样导致连接数量会非常大。因此，为了保证AS内所有的路由器都能学习到完整的BGP路由信息，解决方案：
* 路由反射器
* BGP联邦

这一部分，讲解路由反射器

RR就像一面镜子，将自己学习到的IBGP路由“反射”出去，使得IBGP路由得以在AS内传递。

将一台BGP路由器指定为RR时，还需要指定client。至于client本身，无需做任何配置，而且它不知道网络中RR的存在

### 对等体之间的关系
* Client只需维护与RR之间的IBGP会话
* RR与RR之间需要建立IBGP的全互联
* Non-Client与Non-Client之间需要建立IBGP全互联
* RR与Non-CLient之间需要建立IBGP全互联

### 路由反射宣告原则

当RR收到BGP对等体发来的路由，首先使用BGP选路策略来选择最佳路由。RR在发布学习到的路由信息时，按照RFC2796中的规则发布路由:
* 从非客户机IBGP对等体学习到的路由，**反射**给此RR的所有客户机
* 从客户机学到的路由，**反射**给此RR的所有非客户机和客户机（包括EBGP邻居，不包括发此路由的客户机）
* 从EBGP对等体学到的路由，**发送**给所有的非客户机和客户机
> "反射”和“发送"的区别。“发送”指的是传统情况下（RR不存在的情境下）的BGP路由传递行为。“反射”指的是遵循路由反射规则的情况下，RR执行路由传递动作，被反射出去的路由会被RR插入特殊的路径属性。
 

## RR场景下的BGP路由防环

由于AS_PATH属性在AS内不会发生改变，因此AS内才需要IBGP水平分割用于防止IBGP路由的环路，而RR的存在实际上放宽了水平分割原则。这就会给路由环路带来一定的隐患。BGP通过两个特殊的属性来实现RR场景下的BGP路由环路
* Originator_ID
* Cluster_List

来防止环路，这两个是可选非传递属性

Originator_ID

Originator_ID属性类型为9，是一个32bit的值。RR将一条BGP路由进行反射时会在反射出去的路由中增加该路径属性。
> 1. 如果路由为本地AS始发，则Originator_ID被设置为BGP路由宣告者的Router-ID
> 2. 如果路由非本地AS始发，则Originator_ID被设置为本地AS的边界路由器的Router-ID

如果AS内存在多个RR，则Originator_ID属性由第一个RR创建，并且不会被后续的RR所修改。

当BGP路由器收到一条携带Originator_ID属性的IBGP路由器，并且Originator_ID属性值与自身的router-ID相同，则会忽略关于该路由的更新

Originator_ID及Cluster_list属性将会影响BGP路径优选决策

举个例子：

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc35rn04qlj30hw07qq5a.jpg)

防环示意图

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc35ttq1ouj30do09xad2.jpg)


Cluster

路由反射簇包括反射器RR以及Client，一个AS内允许存在多个cluster。

每一个簇都有唯一的cluster-id。 缺省时是RR的BGP的router-id

当一条路由被反射后，该RR（该簇）的cluster-id就会被添加到路由的cluster-id中

当RR收到一条携带cluster-list属性的BGP路由，且该属性值中包含该簇的cluster-id时，RR认为该条路由存在环路，因此将忽略对该条路由的更新


实例：

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc364d6zugj30gl0astce.jpg)


### 实际配置


比如R4是配置成RR，R3是R4的client

```
R4：

peer 3.3.3.3 reflect-client#设置R3为自己的client


cluster id也是可以修改的
reflector cluster-id id号

```

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc38poav1tj30fr07n0va.jpg)


## BGP联邦（Confederation）
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc38ugmmhoj30iu0aydkj.jpg)

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gc390d21dyj30og0ctgt6.jpg)























