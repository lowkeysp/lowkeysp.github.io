---
title: OTN原理
date: 2019-07-16 09:25:49
tags: [光纤,OTN]
categories: 光纤
mathjax: true
---


## OTN产生的背景

OTN（光传送网，Optical Transport Network）出现之前，主要采用WDM技术实现大容量的传送，但WDM只是物理层面的标准（只针对系统中传送信号的波段、频率间隔等做了规定，而对信号的帧结构没有统一的标准 ），这使得WDM系统中同时传送多种体制的信号（如STM-64，10GbE等），这些信号的性能、帧结构、开销等各不相同，不能方便地进行统一的调度、运营、管理和维护。

## OTN的技术特点

* OTN从其功能上看，在子网内以全光形式传输，而在子网的边界处采用光-电-光转换，各个子网可以通过3R再生器联接，从而构成一个大的光网络。

* 光信号的处理可以基于单个波长，或基于一个波分复用组。在光域内可以实现业务信号的传递、复用、路由选择，支持多种上层业务或协议。

* 在OTN网络的边缘，不同体制的信号被统一封装进入开销丰富的OTN的帧结构中去，并以此为基础，实现基于OTN体制的全网统一运营、管理、维护，便于故障定位，能够提供很好的网络生存性。

* OTN可以解决传统WDM网络波长/子波长业务调度能力弱、组网能力弱（以点到点连接为主的组网方式）、保护能力弱等问题。

* OTN解决了SDH基于VC-12/VC4的交叉颗粒偏小、调度较复杂、不适应大颗粒业务传送需求的问题。




## OTN网络结构

OTN的网络分成可以分为光通道层（OCh，Optical Channel ），光复用段层（OMS，Optical Multiplex Section）和光传送段层（OTS，Optical Transmission Section）三个层面。另外，为了解决客户信号的数字监视问题，光通道层又分为光通路净荷单元（OPU），光通道数据单元（ODUk）和光通道传送单元（OTUk）三个子层，类似于SDH技术的段层和通道层。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g52ty37fhlj30o50dx75a.jpg)


OTN层次结构和信息流之间的关系如下图

![](http://ww1.sinaimg.cn/large/006eDJDNly1g52u08t67xj30r70jdtgr.jpg)


其中，client是光层。OH是overhead 开销的意思。

* OPU（Optical Channel Payload Unit）：光通道净荷单元，提供客户信号的映射功能；
* ODU（Optical Channel Data Unit）：光通道数据单元，提供客户信号的数字包封、OTN的保护倒换、提供踪迹监测、通用通信处理等功能；
* OTU（Optical Channel Transport Unit）：光通道传输单元、提供OTN成帧、FEC处理、通信处理等功能。波分设备中的发送OTU单板完成了信号从客户接口到OCC的变化；波分设备中的接收OTU单板完成了信号从OCC到客户接口的变化。 
* OCC（Optical Channel Carrier）：光通路载波  
* OSC（Optical Supervisory Channel）：光监控信道
* OPS（Optical Physical Section）：光物理段
* OOS（OTM Overhead Signal）：OTM开销信号
* OTM-n.m：n代表系统波长数目，m代表速率级别，m可以是1，2，3，12，23，123。比如23代表一个合路信号当中有10G和40G混和传输的情况。
* OTM-nr.m：中的r代表简化功能。这一接口一般用于没有光监控的系统
* OTM-0.m：代表的是黑白光口，即1310nm或者1550nm的单路光信号。这种接口代表客户侧支持OTN帧结构的信号。


## OTN的映射和复用关系

![](http://ww1.sinaimg.cn/large/006eDJDNly1g53pfg8gg6j30ny0gaqcc.jpg)


用户信号映射到低阶OPU，标识为OPU（L）；OPU（L）信号映射到相关的低阶ODU，标识为ODU（L）；ODU（L）信号映射到相关的OTU[V]信号或者ODTU信号。ODTU信号复用到ODTU组（ODTUG）。ODTUG信号映射到高阶OPU，标识为OPU（H）。OPU（H）信号映射到相关的高阶ODU，标识为ODU（H）。ODU（H）信号映射到相关的OTU[V]。

OPU（L）和OPU（H）具有相同的信息结构，但承载不同的用户信号。ODU（L）和ODU（H）具有相同的信息结构，但承载不同的用户信号。


同SDH类似，用户信号映射到OPUk，一般采用异步映射方法，需要进行速率调整，包括正速率调整和负速率调整。对于复用，OTN采用字节间插式时分复用与波分复用相结合的方法，通过多次时分复用形成OCh，再通过波分复用形成OTM。


对于ODUk的时分复用，4个ODU1信号时分复用到1个ODTUG2，ODTUG2映射到OPU2。ODU2和ODU1的混合信号可以时分复用到ODTUG3，ODTUG3映射到OPU3。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g53zk8n3iaj30o50gzt9y.jpg)

上图表示将4个ODU1信号复用到OPU2信号。1个ODU1信号使用帧定位开销进行扩展，并且通过判决开销（JOH）异步映射光通路数据支路单元1到2（ODTU12）。4个ODTU12信号时分复用到光通路数据单元支路单元群2（ODTUG2），之后映射到OPU2。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g53zps8e9vj30nt0gpadu.jpg)

上面这个图可以更直观的看清楚，客户信号可以是STM-16，ATM，GFP等，组建成一个OPU1，然后再加上一个ODU1开销，就组成了一个ODU1，然后，我们可以将4个ODU1信号复用到一个OPU2，加上一个OPU2开销，之后，再加上一个ODU2开销，就变成了一个ODU2，之后再加上一个OUT2 FEC，就组成了一个OTU2。


下面是另一种ODUk时分复用

![](http://ww1.sinaimg.cn/large/006eDJDNly1g54utb54v7j30qs0f440u.jpg)

表示将最多16个ODU1信号和/或最多4个ODU2信号复用到OPU3信号中。ODU1信号使用帧定位开销进行扩展，并且通过判决开销（JOH）异步映射光通路数据支路单元1到3（ODTU13）。ODU2信号使用帧定位开销进行扩展，并且通过判决开销（JOH）异步映射光通路数据支路单元2到3（ODTU23）。在信号映射到OPU3后，“x”个ODTU23（0≤x≤4）和“16-4x”个ODTU13信号时分复用到光通路数据单元支路单元群3（ODTUG3）。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g54v31z2vwj30of0gkq4z.jpg)

OTU[V]电信号被转换到光通路信号（标识为OCh和OChr）或者OTLk.n。OCh/OChr信号被调制到光通路载波，标识为“OCC和OCCr”。OCC/OCCr信号复用到光载波群，标识为OCG-n.m或OCG-nr.m。OCG-n.m信号加上光复用段开销组成OMSn。OMSn信号加上光传送段开销组成OTSn。OTSn信号出现在OTM-n.m接口。OCG-nr.m信号映射到OPSn。OPSn信号出现在OTM-nr.m接口。单个OCCr信号映射到OPS0。OPS0信号出现在OTM-0.m接口。一组n路OPS0信号出现在OTM-0.mvn接口。

目前为IrDI定义了三种简化功能的OTM接口OTM-nr.m、OTM-0.m和OTM-0.mvn：
* OTM-nr.m支持多波长光通路，信号包含n个OCC，m=1,2,3,4,12,23,34,123,1234，不需要OSC和OOS。
* OTM-0.m支持单波长光通路，m=1,2,3,4，不需要OSC和OOS。
* OTM-0.mvn支持一个多通道光信号。目前定义了2种OTM-0.mvn接口信号：OTM-0.3v4（承载OTU3）和OTM-0.4v4（承载OTU4）。每种承载包含一个OTUk[V]信号分发到四个光通道上的四路光信号。不需要OSC和OOS。

除了上面的三种简化功能的OTM接口，还定义了一种全功能的OTM接口。OTM-n.m：
* OTM-n.m支持单个或多个光区段内的n个光通路，接口不要求3R再生。信号包含n个OCC，m=1,2,3,4,12,23,34,123,1234。



## OTN的帧结构

![](http://ww1.sinaimg.cn/large/006eDJDNly1g54x9nnedaj30ow0dowf5.jpg)

* 帧定界开销用于指示一个帧的开始；
* OTUk开销产生于电光转换之前，用于监控整个光通道的传输性能。
* ODUk是基本的电交叉颗粒，ODUk部分的开销用于监控交叉颗粒的传输性能；
* OPUk用来指示净荷中的业务类型以及进行简单的正负字节调整。
* FEC部分用来对信息净荷部分进行纠错。

OTU帧根据速率等级分为OTU1、OTU2、OTU3、OTU4。OTUk（k=1、2、3、4）帧由OTUk开销、ODUk帧和OTUk FEC三部分组成，总共4行4080列，4×4080=16320字节。





















