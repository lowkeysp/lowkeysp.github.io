---
title: VLAN
date: 2019-05-23 09:45:10
tags: [网络,VLAN]
categories: 网络
---


# VLAN概述
以太网是一种基于CSMA/CD（Carrier Sense Multiple Access/Collision Detect，载波侦听多路访问/冲突检测）的共享通讯介质的数据网络通讯技术，当主机数目较多时会导致冲突严重、广播泛滥、性能显著下降甚至使网络不可用等问题。通过交换机实现LAN互联虽然可以解决冲突（Collision）严重的问题，但仍然不能隔离广播报文。在这种情况下出现了VLAN（Virtual Local Area Network，虚拟局域网）技术，这种技术可以把一个LAN划分成多个逻辑的 LAN——VLAN，每个VLAN是一个广播域，VLAN内的主机间通信就和在一个LAN内一样，而VLAN间则不能直接互通，这样，广播报文被限制在一个VLAN内，如下图所示

![](http://ww1.sinaimg.cn/large/006eDJDNly1g3b0jpahrlj30aj07sdg4.jpg)

VLAN的划分不受物理位置的限制：不在同一物理位置范围的主机可以属于同一个VLAN；一个VLAN包含的用户可以连接在同一个交换机上，也可以跨越交换机，甚至可以跨越路由器。

VLAN的优点如下：
* 限制广播域。广播域被限制在一个VLAN内，节省了带宽，提高了网络处理能力。
* 增强局域网的安全性。VLAN间的二层报文是相互隔离的，即一个VLAN内的用户不能和其它VLAN内的用户直接通信，如果不同VLAN要进行通信，则需通过路由器或三层交换机等三层设备。
* 灵活构建虚拟工作组。用VLAN可以划分不同的用户到不同的工作组，同一工作组的用户也不必局限于某一固定的物理范围，网络构建和维护更方便灵活。


# VLAN原理

要使网络设备能够分辨不同VLAN的报文，需要在报文中添加标识VLAN的字段。
由于普通交换机工作在OSI模型的数据链路层，只能对报文的数据链路层封装进行识别。因此，如果添加识别字段，也需要添加到数据链路层封装中。

IEEE于1999年颁布了用以标准化VLAN实现方案的IEEE802.1Q协议标准草案，对带有 VLAN 标识的报文结构进行了统一规定。

传统的以太网数据帧在目的MAC地址和源MAC地址之后封装的是上层协议的类型
字段，如下图所示。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g3b0xxjnymj30dm021t8m.jpg)

其中DA表示目的MAC地址，SA表示源MAC地址，Type表示报文所属协议类型。IEEE 802.1Q 协议规定在目的MAC地址和源MAC地址之后封装4个字节的VLAN Tag，用以标识 VLAN 的相关信息。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g3b0zo13jbj30c703fdft.jpg)

如上图所示，VLAN Tag包含四个字段，分别是TPID（Tag Protocol Identifier，标签协议标识符）、Priority、CFI（Canonical Format Indicator，标准格式指示位）和VLAN ID。
* TPID用来判断本数据帧是否带有VLAN Tag，长度为16bit，缺省取值为0x8100。
* Priority表示报文的802.1P优先级，长度为3bit，相关内容请参见“QoS 分册”中的“QoS 配置”。
* CFI字段标识MAC地址在不同的传输介质中是否以标准格式进行封装，长度为1bit，取值为0表示MAC地址以标准格式进行封装，为1表示以非标准格
式封装，缺省取值为0。
* VLAN ID标识该报文所属VLAN的编号，长度为12bit，取值范围为0～4095。由于0和4095为协议保留取值，所以VLAN ID的取值范围为1～4094。网络设备利用 VLAN ID来识别报文所属的VLAN，根据报文是否携带VLAN Tag以及携带的 VLAN Tag值，来对报文进行处理。



# 参考
[参考文献](http://www.h3c.com/cn/d_200805/605887_30003_0.htm)
