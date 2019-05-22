---
title: BGP协议
date: 2019-05-22 09:25:03
tags: [路由协议,网络]
categories: 网络
---




# 介绍
## 定义
BGP（Border Gateway Protocol）是一种用于自治系统AS（Autonomous System）之间的动态路由协议。

早期发布的三个版本分别是BGP-1（RFC1105）、BGP-2（RFC1163）和BGP-3（RFC1267），主要用于交换AS之间的可达路由信息，构建AS域间的传播路径，防止路由环路的产生，并在AS级别应用一些路由策略。

当前使用的版本是BGP-4（RFC4271）。

BGP作为事实上的Internet外部路由协议标准，被广泛应用于ISP（Internet Service Provider）之间。

BGP协议具有如下特点：
* BGP是一种外部网关协议（EGP），与OSPF、RIP等内部网关协议（IGP）不同，其着眼点不在于自动发现网络拓扑，而在于在AS之间选择最佳路由和控制路由的传播。

* BGP使用TCP作为其传输层协议（监听端口号为179），提高了协议的可靠性。
  * BGP进行域间的路由选择，对协议的稳定性要求非常高。因此用TCP协议的高可靠性来保证BGP协议的稳定性。
  * BGP的对等体之间必须在逻辑上连通，并进行TCP连接。目的端口号为179，本地端口号任意。
* BGP支持无类别域间路由CIDR（Classless Inter-Domain Routing）。
* 路由更新时，**BGP只发送更新的路由**，大大减少了BGP传播路由所占用的带宽，适用于在Internet上传播大量的路由信息。
* BGP是一种距离矢量（Distance-Vector）路由协议。
* BGP从设计上避免了环路的发生。
  * AS之间：BGP通过携带AS路径信息来标记途经的AS，带有本地AS号的路由将被丢弃，从而避免了域间产生环路。
  * AS内部：BGP在AS内学到的路由不再通告给AS内的BGP邻居，避免了AS内产生环路。
* BGP提供了丰富的路由策略，能够对路由实现灵活的过滤和选择。
* BGP提供了防止路由振荡的机制，有效提高了Internet网络的稳定性。
* BGP易于扩展，能够适应网络新的发展。

## 目的

BGP用于在AS之间传递路由信息，并不是所有情况都需要运行BGP。

![BGP应用场景](http://ww1.sinaimg.cn/large/006eDJDNly1g39uioo0d2j30b709vq3d.jpg)

以下情况中需要使用BGP协议：
* 如图所示，用户需要同时与两个或者多个ISP相连，ISP需要向用户提供部分或完全的Internet路由。这时可以通过BGP路由携带的AS信息来决定到达目的地，走哪一个ISP的AS更为经济
* 不同组织下的用户之间需要传递AS路径信息。
* 用户需要传播组播路由构造组播拓扑，请参见《Quidway S5700 特性描述-IP组播》。

以下情况不需要使用BGP协议：
* 用户只与一个ISP相连
* ISP不需要向用户提供Internet路由。
* AS间使用了缺省路由进行连接

# 原理描述
## 协议基本原理
### BGP运行方式
BGP在交换机上以下列两种方式运行，如上面的图所示
* IBGP（Internal BGP）
* EBGP（External BGP）
当BGP运行于同一AS内部时，被称为IBGP；当BGP运行于不同AS之间时，称为EBGP。
### BGP消息中的角色
* Speaker：发送BGP消息的交换机称为BGP发言者（Speaker），它接收或产生新的路由信息，并发布（Advertise）给其它BGP Speaker。当BGP Speaker收到来自其它AS的新路由时，如果该路由比当前已知路由更优、或者当前还没有该路由，它就把这条路由发布给所有其他BGP Speaker（发送这条路由的BGP Speaker除外）。
* Peer：相互交换消息的BGP Speaker之间互称对等体（Peer），若干相关的对等体可以构成对等体组（Peer Group）。

### BGP的消息
BGP的运行是通过消息驱动的，共有Open、Update、Notification、Keepalive和Route-Refresh等5种消息类型。
* Open消息：是TCP连接建立后发送的第一个消息，用于建立BGP对等体之间的连接关系。对等体在接收到Open消息并协商成功后，将发送Keepalive消息确认并保持连接的有效性。确认后，对等体间可以进行Update、Notification、Keepalive和Route-Refresh消息的交换。
* Update消息：用于在对等体之间交换路由信息。Update消息可以发布多条属性相同的可达路由信息，也可以撤销多条不可达路由信息。
  * 一条Update消息可以发布多条具有相同路由属性的可达路由，这些路由可共享一组路由属性。所有包含在一个给定的Update消息里的路由属性适用于该Update消息中的NLRI（Network Layer Reachability Information）字段里的所有目的地（用IP前缀表示）。
  * 一条Update消息可以撤销多条不可达路由。每一个路由通过目的地（用IP前缀表示），清楚的定义了BGP Speaker之间先前通告过的路由。
  * 一条Update消息可以只用于撤销路由，这样就不需要包括路径属性或者NLRI。相反，也可以只用于通告可达路由，就不需要携带撤销路由信息了。
* Notification消息：当BGP检测到错误状态时，就向对等体发出Notification消息，之后BGP连接会立即中断
* Keepalive消息：BGP会周期性的向对等体发出Keepalive消息，用来保持连接的有效性。
* Route-Refresh消息：Route-Refresh消息用来通知对等体自己支持路由刷新能力（Route-Refresh capability）。
  
  在所有BGP交换机使能Route-Refresh能力的情况下，如果BGP的入口路由策略发生了变化，本地BGP交换机会向对等体发布Route-Refresh消息，收到此消息的对等体会将其路由信息重新发给本地BGP交换机。这样，可以在不中断BGP连接的情况下，对BGP路由表进行动态刷新，并应用新的路由策略。


### BGP有限状态机
BGP有限状态机共有六种状态，分别是Idle、Connect、Active、OpenSent、OpenConfirm和Established。
* Idle状态下，BGP拒绝任何进入的连接请求，是BGP初始状态。
* Connect状态下，BGP等待TCP连接的建立完成后再决定后续操作。
* Active状态下，BGP将尝试进行TCP连接的建立，是BGP的中间状态。
* OpenSent状态下，BGP等待对等体的Open消息。
* OpenConfirm状态下，BGP等待一个Notification报文或Keepalive报文。
* Established状态下，BGP对等体间可以交换Update报文、Route-Refresh报文、Keepalive报文和Notification报文。

在BGP对等体建立的过程中，通常可见的三个状态是：Idle、Active、Established。

BGP对等体双方的状态必须都为Established，BGP邻居关系才能成立，双方通过Update报文交换路由信息

### BGP处理过程

* 因为BGP的传输层协议是TCP协议，所以在BGP对等体建立之前，对等体之间首先进行TCP连接。BGP邻居间会通过Open消息协商相关参数，建立起BGP对等体关系。
* 建立连接后，BGP邻居之间交换整个BGP路由表。BGP协议不会定期更新路由表，但当BGP路由发生变化时，会通过Update消息增量地更新路由表。
* BGP会发送Keepalive消息来维持邻居间的BGP连接。当BGP检测到网络中的错误状态时（例如：收到不支持的协商能力或者收到错误报文时），BGP会发送Notification消息进行报错，BGP连接会随即中断。

### BGP属性
BGP路由属性是一套参数，它对特定的路由进一步的描述，使得BGP能够对路由进行过滤和选择。事实上，所有的BGP路由属性都可以分为以下4类：
* 公认必须遵循的（Well-known mandatory）：所有BGP交换机都可以识别，且必须存在于Update消息中。如果缺少这种属性，路由信息就会出错。
* 公认任意（Well-known discretionary）：所有BGP交换机都可以识别，但不要求必须存在于Update消息中，可以根据具体情况来选择。
* 可选过渡（Optional transitive）：在AS之间具有可传递性的属性。BGP交换机可以不支持此属性，但它仍然会接收这类属性，并传递给其他对等体。
* 可选非过渡（Optional non-transitive）：如果BGP交换机不支持此属性，则相应的这类属性会被忽略，且不会传递给其他对等体。

下面介绍几种常用的BGP路由属性：
* Origin属性

  Origin属性用来定义路径信息的来源，标记一条路由是怎么成为BGP路由的。它有以下3种类型：
  * IGP：具有最高的优先级。通过路由始发AS的IGP得到的路由信息，比如通过network命令注入到BGP路由表的路由，其Origin属性为IGP。
  * EGP：优先级次之。通过EGP得到的路由信息，其Origin属性为EGP。
  * Incomplete：优先级最低。通过其他方式学习到的路由信息。比如BGP通过import-route命令引入的路由，其Origin属性为Incomplete。

* AS_Path属性

  AS_Path属性按矢量顺序记录了某条路由从本地到目的地址所要经过的所有AS编号。

  当BGP Speaker本地通告一条路由时：
    * 当BGP Speaker将这条路由通告到其他AS时，便会将本地AS号添加在AS_Path列表中，并通过Update消息通告给邻居交换机。
    * 当BGP Speaker将这条路由通告到本地AS时，便会在Update消息中创建一个空的AS_Path列表。

  当BGP Speaker传播从其他BGP Speaker的Update消息中学习到的路由时：
    * 当BGP Speaker将这条路由通告到其他AS时，便会把本地AS编号添加在AS_Path列表的最前面（最左面）。收到此路由的BGP交换机根据AS_Path属性就可以知道去目的地址所要经过的AS。离本地AS最近的相邻AS号排在前面，其他AS号按顺序依次排列。
    * 当BGP Speaker将这条路由通告到本地AS时，不会改变这条路由相关的AS_Path属性。

* Next_Hop属性

  BGP的下一跳属性和IGP的有所不同，不一定就是邻居交换机的IP地址。通常情况下，Next_Hop属性遵循下面的规则：
  * BGP Speaker在向EBGP对等体发布某条路由时，会把该路由信息的下一跳属性设置为本地与对端建立BGP邻居关系的接口地址。
  * BGP Speaker将本地始发路由发布给IBGP对等体时，会把该路由信息的下一跳属性设置为本地与对端建立BGP邻居关系的接口地址。
  * BGP Speaker在向IBGP对等体发布从EBGP对等体学来的路由时，并不改变该路由信息的下一跳属性。

* MED

  MED（Multi-Exit-Discriminator）属性仅在相邻两个AS之间传递，收到此属性的AS一方不会再将其通告给任何其他第三方AS。

  MED属性相当于IGP使用的度量值（Metrics），它用于判断流量进入AS时的最佳路由。当一个运行BGP的交换机通过不同的EBGP对等体得到目的地址相同但下一跳不同的多条路由时，在其它条件相同的情况下，将优先选择MED值较小者作为最佳路由。

* Local_Pref属性

  Local_Pref属性仅在IBGP对等体之间有效，不通告给其他AS。它表明交换机的BGP优先级。

  Local_Pref属性用于判断流量离开AS时的最佳路由。当BGP交换机通过不同的IBGP对等体得到目的地址相同但下一跳不同的多条路由时，将优先选择Local_Pref属性值较高的路由。


### BGP选择路由的策略
当到达同一目的地存在多条路由时，BGP采取如下策略进行路由选择：

1. 优选协议首选值（PrefVal）最高的路由。
  
   协议首选值（PrefVal）是华为设备的特有属性，该属性仅在本地有效。

2. 优选本地优先级（Local_Pref）最高的路由。
   
   如果路由没有本地优先级，BGP选路时将该路由按缺省的本地优先级100来处理。通过执行default local-preference命令可以修改BGP路由的缺省本地优先级。

3. 优选本地生成的路由（本地生成的路由优先级高于从邻居学来的路由）。
   
   本地生成的路由包括通过network命令或import-route命令引入的路由、手动聚合路由和自动聚合路由。
   
       a. 优选聚合路由（聚合路由优先级高于非聚合路由）
       b. 通过aggregate命令生成的手动聚合路由的优先级高于通过summary automatic命令生成的自动聚合路由。
       c. 通过network命令引入的路由的优先级高于通过import-route命令引入的路由。
4. 优选AS路径（AS_Path）最短的路由。
   * AS_Path的长度不包括AS_CONFED_SEQUENCE和AS_CONFED_SET。
   * AS_SET的长度为1，无论AS_SET中包括多少AS号。
   * 执行bestroute as-path-ignore命令后，BGP选路时，忽略AS_Path的比较。
5. 比较Origin属性，依次优选Origin类型为IGP、EGP、Incomplete的路由。
6. 优选从EBGP邻居学来的路由（EBGP路由优先级高于IBGP路由）。

   依次优选EBGP路由、IBGP路由、LocalCross路由、RemoteCross路由。
   
   PE上某个VPN实例的VPNv4路由的ERT匹配其他VPN实例的IRT后复制到该VPN实例，称为LocalCross；从远端PE学习到的VPNv4路由的ERT匹配某个VPN实例的IRT后复制到该VPN实例，称为RemoteCross。
7. 优选到BGP下一跳IGP Metric较小的路由。
      
         说明：如果配置了负载分担，当上述所有规则相同，且存在多条As_Path完全相同的外部路由，则根据配置的路由条数选择多条路由进行负载分担。

8. 优选Cluster_List最短的路由。
9. 优选Router ID最小的交换机发布的路由。
         
         说明：如果路由携带Originator_ID属性，选路过程中将比较Originator_ID的大小（不再比较Router ID），并优选Originator_ID最小的路由。
10. 比较对等体的IP Address，优选从具有较小IP Address的对等体学来的路由。

### BGP等价负载分担
当到达同一目的地址存在多条等价路由时，可以通过BGP等价负载分担实现均衡流量的目的。

形成BGP等价负载分担的条件是：“BGP选择路由的策略”的1至8条规则中需要比较的属性完全相同。

### BGP发布路由的策略
BGP发布路由时采用如下策略：
* 存在多条有效路由时，BGP Speaker只将最优路由发布给对等体。
* BGP Speaker从EBGP获得的路由会向它所有BGP对等体发布（包括EBGP对等体和IBGP对等体）。
* BGP Speaker从IBGP获得的路由不向它的IBGP对等体发布。
* BGP Speaker从IBGP获得的路由发布给它的EBGP对等体。
* 连接一旦建立，BGP Speaker将把自己所有BGP路由发布给新对等体。

### IBGP和IGP同步
同步是指IBGP和IGP之间的同步，其目的是避免误导外部AS的交换机。

如果一个AS中有非BGP交换机提供转发服务，经该AS转发的IP报文将可能因为目的地址不可达而被丢弃。如图所示，SwitchE通过BGP从SwitchD可以学到SwitchA的一条路由8.0.0.0/8，于是将到这个目的地址的报文转发给SwitchD，SwitchD查询路由表，发现下一跳是SwitchB。由于SwitchD从IGP学到了到SwitchB的路由，所以通过路由迭代，SwitchD将报文转发给SwitchC。但SwitchC并不知道去8.0.0.0/8的路由，于是将报文丢弃。

![IBGP和IGP同步](http://ww1.sinaimg.cn/large/006eDJDNly1g39wq1u8c5j30e804imx4.jpg)

如果设置了同步特性，在IBGP路由加入路由表并发布给EBGP对等体之前，会先检查IGP路由表。只有在IGP也知道这条IBGP路由时，它才会被加入到路由表，并发布给EBGP对等体。

在下面的情况中，可以安全地关闭同步特性。

* 本AS不是过渡AS（图中的AS20就属于一个过渡AS）
* 本AS内所有交换机建立IBGP全连接

## 路由引入
BGP协议自身不能发现路由，所以需要引入其他协议的路由（如IGP或者静态路由等）注入到BGP路由表中，从而将这些路由在AS之内和AS之间传播。

BGP引入路由时支持Import和Network两种方式：

* Import方式是按协议类型，将RIP路由、OSPF路由、ISIS路由、静态路由和直连路由等某一协议的路由注入到BGP路由表中。
* Network方式比Import方式更精确，将指定前缀和掩码的一条路由注入到BGP路由表中。

## 路由聚合

在大规模的网络中，BGP路由表十分庞大，使用路由聚合（Routes Aggregation）可以大大减小路由表的规模。

路由聚合实际上是将多条路由合并的过程。这样BGP在向对等体通告路由时，可以只通告聚合后的路由，而不是通告所有的具体路由。

BGP路由聚合支持两种方式：

* 自动聚合：对BGP引入的路由进行聚合。配置自动聚合后，对参加聚合的具体路由进行抑制，BGP将按照自然网段聚合路由（如10.1.1.1/24和10.2.1.1/24将聚合为A类地址10.0.0.0/8），并且BGP向对等体只发送聚合后的路由。
* 手动聚合：对BGP本地路由进行聚合。手动聚合可以控制聚合路由的属性，以及决定是否发布具体路由。

IPv4支持自动聚合和手动聚合两种方式。

## 路由反射器
为保证IBGP对等体之间的连通性，需要在IBGP对等体之间建立全连接（Full-mesh）关系。假设在一个AS内部有n台交换机，那么应该建立的IBGP连接数就为n(n-1)/2。当IBGP对等体数目很多时，对网络资源和CPU资源的消耗都很大。利用路由反射可以解决这一问题。

在一个AS内，其中一台交换机作为路由反射器RR（Route Reflector），其它交换机作为客户机（Client）。客户机与路由反射器之间建立IBGP连接。路由反射器和它的客户机组成一个集群（Cluster）。路由反射器在客户机之间反射路由信息，客户机之间不需要建立BGP连接。

既不是反射器也不是客户机的BGP设备被称为非客户机（Non-Client）。非客户机与路由反射器之间，以及所有的非客户机之间仍然必须建立全连接关系。如图所示。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g39wyz8caej30a406lq34.jpg)


### 应用
当RR收到对等体发来的路由，首先使用BGP选路策略来选择最佳路由。在向IBGP邻居发布学习到的路由信息时，RR按照RFC2796中的规则发布路由。
* 从非客户机IBGP对等体学到的路由，发布给此RR的所有客户机。
* 从客户机学到的路由，发布给此RR的所有非客户机和客户机（发起此路由的客户机除外）。
* 从EBGP对等体学到的路由，发布给所有的非客户机和客户机。

RR的配置方便，只需要对作为反射器的交换机进行配置，客户机并不需要知道自己是客户机。

在某些网络中，路由反射器的客户机之间已经建立了全连接，它们可以直接交换路由信息，此时客户机到客户机之间的路由反射是没有必要的，而且还占用带宽资源。S5700支持配置命令undo reflect between-clients来禁止客户机之间的路由反射，但客户机到非客户机之间的路由仍然可以被反射。缺省情况下，允许客户机之间的路由反射。

### Originator_ID属性

RFC2796定义了Originator_ID属性和Cluster_List属性，用于检测和防止路由环路。

Originator_ID属性长4字节，由路由反射器（RR）产生，携带了本地AS内部路由发起者的Router ID。
* 当一条路由第一次被RR反射的时候，RR将Originator_ID属性加入这条路由，标识这条路由的发起交换机。如果一条路由中已经存在了Originator_ID属性，则RR将不会创建新的Originator_ID。
* 当其他BGP Speaker接收到这条路由的时候，将比较收到的Originator_ID和本地的Router ID，如果两个ID相同，BGP Speaker会忽略掉这条路由，不做处理。

### Cluster_List属性
对于AS之间，BGP用于防止环路的主要措施是通过AS_Path属性记录途经的AS路径，带有本地AS号的路由将被交换机丢弃；对于AS之内，BGP防止路由环路的方法是禁止IBGP对等体发布从AS内部学来的路由。

RR的实现是基于放宽对“BGP在AS内学到的路由不会在AS中转发”的要求，即允许IBGP对等体之间发布从AS内部学来的路由。在这种情况下，Cluster_List属性被引入，用于防止AS内部的环路。

路由反射器和它的客户机组成一个集群（Cluster）。在一个AS内，每个路由反射器使用唯一的CLUSTER_ID作为标识。

为防止产生路由环路，路由反射器使用CLUSTER_LIST，记录反射路由经过的所有CLUSTER_ID。

Cluster_List由一系列的Cluster_ID组成，描述了一条路由所经过的反射器路径，这和描述路由经过的AS路径的AS_Path属性有相似之处。Cluster_List由路由反射器产生。

* 当RR在它的客户机之间或客户机与非客户机之间反射路由时，RR会把本地Cluster_ID添加到Cluster_List的前面。如果Cluster_List为空，RR就创建一个。
* 当RR接收到一条更新路由时，RR会检查Cluster_List。如果Cluster_List中已经有本地Cluster_ID，丢弃该路由；如果没有本地Cluster_ID，将其加入Cluster_List，然后反射该更新路由。

### 备份RR
为增加网络的可靠性，防止单点故障，有时需要在一个集群中配置一个以上的路由反射器。这时，相同集群中的路由反射器要共享相同的Cluster_ID，以避免路由环路。S5700中需要使用命令reflector cluster-id给所有位于同一个集群内的路由反射器配置相同的Cluster_ID。

在冗余的环境里，客户机会收到不同反射器发来的到达同一目的地的多条路由，这时客户机应用BGP选择路由的策略来选择最佳路由。

![备份路由反射器 ](http://ww1.sinaimg.cn/large/006eDJDNly1g39xa0cap3j308v062wef.jpg)

如图，路由反射器RR1和RR2在同一个Cluster内。RR1和RR2之间配置IBGP连接，即两个反射器互为非客户机。
* 当客户机Client1从外部对等体接收到一条更新路由后，它通过IBGP向RR1和RR2通告这条路由。
* RR1接收到该更新路由后，它向其他的客户机（Client2、Client3）和非客户机（RR2）反射，同时将本地Cluster_ID添加到Cluster_List前面。
* RR2接收到该反射路由后，检查Cluster_List，发现自己的Cluster_ID已经包含在Cluster_List中。因此，它丢弃该更新路由，不再向自己的客户机反射。

说明： Cluster_List的应用保证了同一AS内的不同RR之间不出现路由循环。

### AS内多个集群
一个AS中可能存在多个集群（Cluster）。各个RR之间是IBGP对等体的关系，一个RR可以把另一个RR配置成自己的客户机或非客户机。因此可以灵活的配置AS内部集群与集群之间的关系

例如，一个骨干网被分成多个反射集群，每个RR将其它的RR配置成非客户机，各RR之间建立全连接。每个客户机只与所在集群的RR建立IBGP连接。这样该自治系统内的所有BGP交换机都会收到反射路由信息。如图所示
![AS内多个集群](http://ww1.sinaimg.cn/large/006eDJDNly1g39xi8dnf2j30ec09qt8w.jpg)

### 分级反射器

在实际的反射器部署中，常用的是分级反射器的场景。如下图，ISP为AS100提供Internet路由，ISP与AS100内建立双出口EBGP连接。AS100内部分为两个集群。Cluster1内的四台交换机是核心交换机。
* Cluster1中部署了两个一级RR（RR-1），这种冗余结构保证了AS100内部网络核心层的可靠性。核心层其余两台交换机作为RR-1的客户机。
* Cluster2中部署了一个二级RR（RR-2），这个RR-2同时也是RR-1的客户机。

![分级反射器](http://ww1.sinaimg.cn/large/006eDJDNly1g39xlfbfwwj308m08j0sp.jpg)

窍门：反射器场景下，如果BGP的优选路由不需要指导转发，通过配置BGP-RIB-ONLY特性，使所有BGP的优选路由都不加入IP路由表，也不进入转发层，从而提高转发效率和提升系统容量。