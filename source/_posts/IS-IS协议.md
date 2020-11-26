---
title: IS-IS协议
date: 2019-05-21 14:20:10
tags: [路由协议,网络,IS-IS]
categories: 网络
---



<meta name="referrer" content="no-referrer" />



<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->




# 介绍
## 定义
中间系统到中间系统IS-IS（Intermediate System to Intermediate System）属于内部网关协议IGP（Interior Gateway Protocol），用于自治系统内部。IS-IS也是一种链路状态协议，使用最短路径优先SPF（Shortest Path First）算法进行路由计算。

## 目的
IS-IS是国际标准化组织ISO（the International Organization for Standardization）为它的无连接网络协议CLNP（ConnectionLess Network Protocol）设计的一种动态路由协议。

随着TCP/IP协议的流行，为了提供对IP路由的支持，IETF（Internet Engineering Task Force ）在RFC1195中对IS-IS进行了扩充和修改，使它能够同时应用在TCP/IP和OSI（Open System Interconnection）环境中，称为集成IS-IS（Integrated IS-IS或Dual IS-IS）。

# 原理

## IS-IS基本概念
### IS-IS拓扑结构
#### IS-IS整体结构
为了支持大规模的路由网络，IS-IS在自治系统内采用骨干区域与非骨干区域两级的分层结构。一般来说，将Level-1路由器部署在非骨干区域，Level-2路由器和Level-1-2路由器部署在骨干区域。每一个非骨干区域都通过Level-1-2路由器与骨干区域相连。

下图是一个运行IS-IS协议的网络。它与OSPF的多区域网络拓扑结构非常相似。整个骨干区域不仅包括Area1中的所有路由器，还包括其它区域的Level-1-2路由器
![IS-IS拓扑结构](http://ww1.sinaimg.cn/large/006eDJDNly1g38y7hu6atj30c409l3ze.jpg)

除此之外，还有另外一种拓扑结构。如下图，在这个拓扑中，Level-2级别的路由器没有在同一个区域，而是分别属于不同的区域。此时所有物理连续的Level-1-2和Level-2路由器就构成了IS-IS的骨干区域。
![另一种拓扑结构](http://ww1.sinaimg.cn/large/006eDJDNly1g38yfbzhzxj30d5061wew.jpg)

通过以上两种拓扑结构图可以体现IS-IS与OSPF的不同点：
* 在IS-IS中，每个路由器都只属于一个区域；而在OSPF中，一个路由器的不同接口可以属于不同的区域。
* 在IS-IS中，单个区域没有骨干与非骨干区域的概念；而在OSPF中，Area0被定义为骨干区域。
* 在IS-IS中，Level-1和Level-2级别的路由都采用SPF算法，分别生成最短路径树SPT（Shortest Path Tree）；而在OSPF中，只有在同一个区域内才使用SPF算法，区域之间的路由需要通过骨干区域来转发。

#### IS-IS路由器的分类
* Level-1路由器

Level-1路由器负责区域内的路由，它只与属于同一区域的Level-1和Level-1-2路由器形成邻居关系，属于不同区域的Level-1路由器不能形成邻居关系。Level-1路由器只负责维护Level-1的链路状态数据库LSDB（Link State Database），该LSDB包含本区域的路由信息，到本区域外的报文转发给最近的Level-1-2路由器。

* Level-2路由器

Level-2路由器负责区域间的路由，它可以与同一或者不同区域的Level-2路由器或者其它区域的Level-1-2路由器形成邻居关系。Level-2路由器维护一个Level-2的LSDB，该LSDB包含区域间的路由信息。

所有Level-2级别（即形成Level-2邻居关系）的路由器组成路由域的骨干网，负责在不同区域间通信。路由域中Level-2级别的路由器必须是物理连续的，以保证骨干网的连续性。只有Level-2级别的路由器才能直接与区域外的路由器交换数据报文或路由信息。

* Level-1-2路由器

同时属于Level-1和Level-2的路由器称为Level-1-2路由器，它可以与同一区域的Level-1和Level-1-2路由器形成Level-1邻居关系，也可以与其他区域的Level-2和Level-1-2路由器形成Level-2的邻居关系。Level-1路由器必须通过Level-1-2路由器才能连接至其他区域。

#### IS-IS的网络类型
IS-IS只支持两种类型的网络，根据物理链路不同可分为：
* 广播链路：如Ethernet、Token-Ring等。
* 点到点链路：如PPP、HDLC等

#### 说明
>  对于NBMA（Non-Broadcast Multi-Access）网络，如ATM，需对其配置子接口，并注意子接口类型应配置为P2P
>  IS-IS不能在点到多点链路P2MP（Point to MultiPoint）上运行。


#### DIS和伪节点
在广播网络中，IS-IS需要在所有的路由器中选举一个路由器作为DIS（Designated Intermediate System）。DIS用来创建和更新伪节点（Pseudonodes），并负责生成伪节点的链路状态协议数据单元LSP（Link state Protocol Data Unit），用来描述这个网络上有哪些网络设备。

伪节点是用来模拟广播网络的一个虚拟节点，并非真实的路由器。在IS-IS中，伪节点用DIS的System ID和一个字节的Circuit ID（非0值）标识。

下图为伪节点示意图

![伪节点示意图](http://ww1.sinaimg.cn/large/006eDJDNly1g38yyhcotvj30dr079gls.jpg)

从图中可以看出，使用伪节点可以简化网络拓扑，使路由器产生的LSP长度较小。另外，当网络发生变化时，需要产生的LSP数量也会较少，减少SPF的资源消耗。

Level-1和Level-2的DIS是分别选举的，用户可以为不同级别的DIS选举设置不同的优先级。DIS优先级数值最大的被选为DIS。如果优先级数值最大的路由器有多台，则其中MAC地址最大的路由器会被选中。不同级别的DIS可以是同一台路由器，也可以是不同的路由器

IS-IS协议中DIS与OSPF协议中DR（Designated Router）的区别：

* 在IS-IS广播网中，优先级为0的路由器也参与DIS的选举，而在OSPF中优先级为0的路由器则不参与DR的选举。
* 在IS-IS广播网中，当有新的路由器加入，并符合成为DIS的条件时，这个路由器会被选中成为新的DIS，原有的伪节点被删除。此更改会引起一组新的LSP泛洪。而在OSPF中，当一台新路由器加入后，即使它的DR优先级值最大，也不会立即成为该网段中的DR。
* 在IS-IS广播网中，同一网段上的同一级别的路由器之间都会形成邻接关系，包括所有的非DIS路由器之间也会形成邻接关系。而在OSPF中，路由器只与DR和BDR建立邻接关系。

#### 说明
> IS-IS广播网上所有的路由器之间都形成邻接关系，但LSDB的同步仍然依靠DIS来保证。

#### IS-IS的地址结构
网络服务访问点NSAP（Network Service Access Point）是OSI协议中用于定位资源的地址。NSAP的地址结构如下图所示，它由IDP（Initial Domain Part）和DSP（Domain Specific Part）组成。IDP和DSP的长度都是可变的，NSAP总长最多是20个字节，最少8个字节。

![NSAP结构](http://ww1.sinaimg.cn/large/006eDJDNly1g38z7ziicgj30cv04ojra.jpg)
* IDP相当于IP地址中的主网络号。它是由ISO规定，并由AFI（Authority and Format Identifier）与IDI（Initial Domain Identifier）两部分组成。AFI表示地址分配机构和地址格式，IDI用来标识域
* DSP相当于IP地址中的子网号和主机地址。它由High Order DSP、System ID和SEL三个部分组成。High Order DSP用来分割区域，System ID用来区分主机，SEL（NSAP Selector）用来指示服务类型。
* Area Address： IDP和DSP中的High Order DSP一起，既能够标识路由域，也能够标识路由域中的区域，因此，它们一起被称为区域地址（Area Address），相当于OSPF中的区域编号。同一Level-1区域内的所有路由器必须具有相同的区域地址，Level-2区域内的路由器可以具有不同的区域地址。一般情况下，一个路由器只需要配置一个区域地址，且同一区域中所有节点的区域地址都要相同。为了支持区域的平滑合并、分割及转换，在设备的实现中，一个IS-IS进程下最多可配置3个区域地址。
* System ID：System ID用来在区域内唯一标识主机或路由器。在设备的实现中，它的长度固定为48bit（6字节）。在实际应用中，一般使用Router ID与System ID进行对应。假设一台路由器使用接口Loopback0的IP地址168.10.1.1作为Router ID，则它在IS-IS中使用的System ID可通过如下方法转换得到：
  
  * 将IP地址168.10.1.1的每个十进制数都扩展为3位，不足3位的在前面补0，得到168.010.001.001。
  * 将扩展后的地址分为3部分，每部分由4位数字组成，得到1680.1000.1001。重新组合的1680.1000.1001就是System ID。
  
  实际System ID的指定可以有不同的方法，但要保证能够唯一标识主机或路由器。

* SEL: SEL的作用类似IP中的“协议标识符”，不同的传输协议对应不同的SEL。在IP上SEL均为00。

网络实体名称NET（Network Entity Title）指的是设备本身的网络层信息，可以看作是一类特殊的NSAP（SEL＝00）。NET的长度与NSAP的相同，最多为20个字节，最少为8个字节。在路由器上配置IS-IS时，只需要考虑NET即可，NSAP可不必去关注。

例如有NET为：ab.cdef.1234.5678.9abc.00，则其中Area Address为ab.cdef，System ID为1234.5678.9abc，SEL为00。

#### IS-IS的报文类型
IS-IS报文有以下几种类型：HELLO PDU（Protocol Data Unit）、LSP PDU和SNP PDU。
* Hello PDU

  Hello报文用于建立和维持邻居关系，也称为IIH（IS-to-IS Hello PDUs）。其中，广播网中的Level-1 IS-IS使用Level-1 LAN IIH；广播网中的Level-2 IS-IS使用Level-2 LAN IIH；非广播网络中则使用P2P IIH。它们的报文格式有所不同。P2P IIH中相对于LAN IIH来说，多了一个表示本地链路ID的Local Circuit ID字段，缺少了表示广播网中DIS的优先级的Priority字段以及表示DIS和伪节点System ID的LAN ID字段。

* LSP PDU

  链路状态报文LSP（Link State PDUs）用于交换链路状态信息。LSP分为两种：Level-1 LSP和Level-2 LSP。Level-1 LSP由Level-1 IS-IS传送，Level-2 LSP由Level-2 IS-IS传送，Level-1-2 IS-IS则可传送以上两种LSP。

  LSP报文中主要字段的解释如下：
  * ATT字段：当Level-1-2 IS-IS在Level-1区域内传送Level-1 LSP时，如果Level-1 LSP中设置了ATT位，则表示该区域中的Level-1 IS-IS可以通过此Level-1-2 IS-IS通往外部区域。
  * OL（LSDB Overload）字段：过载标志位
  
    设置了过载标志位的LSP虽然还会在网络中扩散，但是在计算通过过载路由器的路由时不会被采用。即对路由器设置过载位后，其它路由器在进行SPF计算时不会使用这台路由器做转发，只计算该节点上的直连路由。更多内容请参见《原理描述-IS-IS过载位》
  * IS Type字段 : 用来指明生成此LSP的IS-IS类型是Level-1还是Level-2 IS-IS（01表示Level-1，11表示Level-2）。

* SNP PDU
  
  序列号报文SNP（Sequence Number PDUs）通过描述全部或部分数据库中的LSP来同步各LSDB（Link-State DataBase），从而维护LSDB的完整与同步。
  
  SNP包括全序列号报文CSNP（Complete SNP）和部分序列号报文PSNP（Partial SNP），进一步又可分为Level-1 CSNP、Level-2 CSNP、Level-1 PSNP和Level-2 PSNP。

  CSNP包括LSDB中所有LSP的摘要信息，从而可以在相邻路由器间保持LSDB的同步。在广播网络上，CSNP由DIS定期发送（缺省的发送周期为10秒）；在点到点链路上，CSNP只在第一次建立邻接关系时发送。

  PSNP只列举最近收到的一个或多个LSP的序号，它能够一次对多个LSP进行确认，当发现LSDB不同步时，也用PSNP来请求邻居发送新的LSP。


IS-IS报文中的变长字段部分是多个TLV（Type-Length-Value）三元组。其格式如下图所示。TLV也称为CLV（Code-Length-Value）。

![TLV格式](http://ww1.sinaimg.cn/large/006eDJDNly1g38zyg3oc9j30a702pt8i.jpg)

不同PDU类型所包含的TLV是不同的。如下表所示
![](http://ww1.sinaimg.cn/large/006eDJDNly1g38zzqkmg5j30h50fhwex.jpg)

### IS-IS基本原理
IS-IS是一种链路状态路由协议，每一台路由器都会生成一个LSP，它包含了该路由器所有使能IS-IS协议接口的链路状态信息。通过跟相邻设备建立IS-IS邻接关系，互相更新本地设备的LSDB，可以使得LSDB与整个IS-IS网络的其他设备的LSDB实现同步。然后根据LSDB运用SPF算法计算出IS-IS路由。如果此IS-IS路由是到目的地址的最优路由，则此路由会下发的IP路由表中，并指导报文的转发。

#### IS-IS邻居关系的建立
两台运行IS-IS的路由器在交互协议报文实现路由功能之前必须首先建立邻居关系。在不同类型的网络上，IS-IS的邻居建立方式并不相同。
* 广播链路邻居关系的建立

  如下图所示，以Level-2路由器为例，描述了广播链路中建立邻居关系的过程。Level-1路由器之间建立邻居与此相同。

  ![ 广播链路邻居关系建立过程](http://ww1.sinaimg.cn/large/006eDJDNly1g3906b67v9j30el08074i.jpg)

  1. RouterA广播发送Level-2 LAN IIH，此报文中无邻居标识。
  2. RouterB收到此报文后，将自己和RouterA的邻居状态标识为Initial。然后，RouterB再向RouterA回复Level-2 LAN IIH，此报文中标识RouterA为RouterB的邻居。
  3. RouterA收到此报文后，将自己与RouterB的邻居状态标识为Up。然后RouterA再向RouterB发送一个标识RouterB为RouterA邻居的Level-2 LAN IIH。
  4. RouterB收到此报文后，将自己与RouterA的邻居状态标识为Up。这样，两个路由器成功建立了邻居关系。
  
  因为是广播网络，需要选举DIS，所以在邻居关系建立后，路由器会等待两个Hello报文间隔，再进行DIS的选举。Hello报文中包含Priority字段，Priority值最大的将被选举为该广播网的DIS。若优先级相同，接口MAC地址较大的被选举为DIS。

* P2P链路邻居关系的建立
  
  在P2P链路上，邻居关系的建立不同于广播链路。分为两次握手机制和三次握手机制
  * 两次握手机制
    
    只要路由器收到对端发来的Hello报文，就单方面宣布邻居为Up状态，建立邻居关系
  * 三次握手机制
    
    此方式通过三次发送P2P的IS-IS Hello PDU最终建立起邻居关系，类似广播邻居关系的建立。

  两次握手机制存在明显的缺陷。当路由器间存在两条及以上的链路时，如果某条链路上到达对端的单向状态为Down，而另一条链路同方向的状态为Up，路由器之间还是能建立起邻接关系。SPF在计算时会使用状态为UP的链路上的参数，这就导致没有检测到故障的路由器在转发报文时仍然试图通过状态为Down的链路。三次握手机制解决了上述不可靠点到点链路中存在的问题。这种方式下，路由器只有在知道邻居路由器也接收到它的报文时，才宣布邻居路由器处于Up状态，从而建立邻居关系。

IS-IS按如下原则建立邻居关系：
* 只有同一层次的相邻路由器才有可能成为邻居。
* 对于Level-1路由器来说，区域号必须一致
* 链路两端IS-IS接口的网络类型必须一致。
* 链路两端IS-IS接口的地址必须处于同一网段。由于IS-IS是直接运行在数据链路层上的协议，并且最早设计是给CLNP使用的，IS-IS邻居关系的形成与IP地址无关。但在实际的实现中，由于只在IP上运行IS-IS，所以是要检查对方的IP地址的。如果接口配置了从IP，那么只要双方有某个IP（主IP或者从IP）在同一网段，就能建立邻居，不一定要主IP相同。

### IS-IS的LSP交互过程
#### LSP产生的原因
IS-IS路由域内的所有路由器都会产生LSP，以下事件会触发一个新的LSP：
* 邻居Up或Down
* IS-IS相关接口Up或Down
* 引入的IP路由发生变化
* 区域间的IP路由发生变化
* 接口被赋了新的metric值
* 周期性更新

#### 收到邻居新的LSP的处理过程
1. 将接收的新的LSP合入到自己的LSDB数据库中，并标记为flooding。
2. 发送新的LSP到除了收到该LSP的接口之外的接口。
3. 邻居再扩散到其他邻居

#### LSP的“泛洪”
LSP报文的“泛洪”（flooding）是指当一个路由器向相邻路由器通告自己的LSP后，相邻路由器再将同样的LSP报文传送到除发送该LSP的路由器外的其它邻居，并这样逐级将LSP传送到整个层次内所有路由器的一种方式。通过这种“泛洪”，整个层次内的每一个路由器就都可以拥有相同的LSP信息，并保持LSDB的同步。

每一个LSP都拥有一个标识自己的4字节的序列号。在路由器启动时所发送的第一个LSP报文中的序列号为1，以后当需要生成新的LSP时，新LSP的序列号在前一个LSP序列号的基础上加1。更高的序列号意味着更新的LSP。

#### 广播链路中新加入路由器与DIS同步LSDB数据库的过程
下面是一个示意图

![](http://ww1.sinaimg.cn/large/006eDJDNly1g390rtholzj30a90b0mxc.jpg)

1. 如图所示，新加入的路由器RouterC首先发送Hello报文，与该广播域中的路由器建立邻居关系。
2. 建立邻居关系之后，RouterC等待LSP刷新定时器超时，然后将自己的LSP发往组播地址（Level-1：01-80-C2-00-00-14；Level-2：01-80-C2-00-00-15）。这样网络上所有的邻居都将收到该LSP。
3. 该网段中的DIS会把收到RouterC的LSP加入到LSDB中，并等待CSNP报文定时器超时并发送CSNP报文，进行该网络内的LSDB同步。
4. RouterC收到DIS发来的CSNP报文，对比自己的LSDB数据库，然后向DIS发送PSNP报文请求自己没有的LSP。
5. DIS收到该PSNP报文请求后向RouterC发送对应的LSP进行LSDB的同步。

在上述过程中DIS的LSDB更新过程如下：
1. DIS接收到LSP，在数据库中搜索对应的记录。若没有该LSP，则将其加入数据库，并广播新数据库内容。
2. 若收到的LSP序列号大于本地LSP的序列号，就替换为新报文，并广播新数据库内容；若收到的LSP序列号小本地LSP的序列号，就向入端接口发送本地LSP报文。
3. 若两个序列号相等，则比较Remaining Lifetime。若收到的LSP的Remaining Lifetime小于本地LSP的Remaining Lifetime，就替换为新报文，并广播新数据库内容；若收到的LSP的Remaining Lifetime大于本地LSP的Remaining Lifetime，就向入端接口发送本地LSP报文。
4. 若两个序列号和Remaining Lifetime都相等，则比较Checksum。若收到的LSP的Checksum大于本地LSP的Checksum，就替换为新报文，并广播新数据库内容；若收到的LSP的Checksum小于本地LSP的Checksum，就向入端接口发送本地LSP报文。
5. 若两个序列号、Remaining Lifetime和Checksum都相等，则不转发该报文。

#### P2P链路上LSDB数据库的同步过程
如图所示
![](http://ww1.sinaimg.cn/large/006eDJDNly1g390z83g39j30a407d3yk.jpg)

1. RouterA先与RouterB建立邻居关系。
2. 建立邻居关系之后，RouterA与RouterB会先发送CSNP给对端设备。如果对端的LSDB与CSNP没有同步，则发送PSNP请求索取相应的LSP。
3. 如图所示假定RouterB向RouterA索取相应的LSP。RouterA发送RouterB请求的LSP的同时启动LSP重传定时器，并等待RouterB发送的PSNP作为收到LSP的确认。
4. 如果在接口LSP重传定时器超时后，RouterA还没有收到RouterB发送的PSNP报文作为应答，则重新发送该LSP直至收到PSNP报文。

在P2P链路上PSNP有两种作用：
* 作为Ack应答以确认收到的LSP。
* 用来请求所需的LSP。

在P2P链路中设备的LSDB更新过程如下：
1. 若收到的LSP比本地的序列号更小，则直接给对方发送本地的LSP，然后等待对方给自己一个PSNP报文作为确认；若收到的LSP比本地的序列号更大，则将这个新的LSP存入自己的LSDB，再通过一个PSNP报文来确认收到此LSP，最后再将这个新LSP发送给除了发送该LSP的邻居以外的邻居。
2. 若收到的LSP序列号和本地相同，则比较Remaining Lifetime，若收到LSP的Remaining Lifetime小于本地LSP的Remaining Lifetime，则将收到的LSP存入LSDB中并发送PSNP报文来确认收到此LSP，然后将该LSP发送给除了发送该LSP的邻居以外的邻居；若收到LSP的Remaining Lifetime大于本地LSP的Remaining Lifetime，则直接给对方发送本地的LSP，然后等待对方给自己一个PSNP报文作为确认。
3. 若收到的LSP和本地LSP的序列号和Remaining Lifetime都相同，则比较Checksum，若收到LSP的Checksum大于本地LSP的Remaining Lifetime，则将收到的LSP存入LSDB中并发送PSNP报文来确认收到此LSP，然后将该LSP发送给除了发送该LSP的邻居以外的邻居；若收到LSP的Checksum小于本地LSP的Remaining Lifetime，则直接给对方发送本地的LSP，然后等待对方给自己一个PSNP报文作为确认。
4. 若收到的LSP和本地LSP的序列号、Remaining Lifetime和Checksum都相同，则不转发该报文。


# 参考

[参考文献1](http://support-trial.huawei.com/enterprise/docinforeader!loadDocument.action?contentId=DOC1000009561&partNo=10032#dc_feature_isis_0000)
