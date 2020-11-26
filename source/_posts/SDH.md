---
title: SDH
date: 2019-07-11 15:48:22
tags: [光纤,SDH]
categories: 光纤
mathjax: true
---


<meta name="referrer" content="no-referrer" />

## 什么是SDH？
SDH是Synchronous Digital Hierarchy 的简称，也就是同步数字体系。是一整套可进行同步数字传输、复用和交叉连接的标准化数字信号的等级结构。

## SDH的工作方式

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4vyap7j37j30zn0gkjtv.jpg)

STM-1是SDH的第一个等级，又叫基本同步传送模块，比特率为155.520Mb/s。STM-N是SDH第N个等级的同步传送模块，比特率是STM-1的N倍(N=4n=1，4，16，64，256)。

低阶SDH  -> 高阶SDH

比如，STM-1->STM-4。采用字节间插复用方式，4个STM-1变成一个STM-4

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4vyf20o6jj30wk0deac2.jpg)


## 帧结构

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4vygod61wj311q0ii44b.jpg)

可以计算，当N=1时，也就是STM-1，$8\times(9\times270\times1)\times8000=155520000=155.52Mbit/s$

### STM-N净负荷

STM-N净负荷是一个9行$\times$ 261列 $\times$N。

这是用来放置各种业务信息的地方。2M/34M/140M等PDH信号、ATM信号、IP信息包等打包成信息包后，放于其中。然后由STM-N信号承载，在SDH网上传输。若将STM-N信号帧比做一辆货车，其净负荷区即为该货车的车厢。

在将低速信号打包装箱时，在每一个信息包中加入通道开销POH，以完成对每一个“货物包”在“运输”中的监视。

下图是对上述描述的解释。低速信号打包后，再加上POH，然后就是一个信息包，放到这个净负荷区中。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x0jy1lp1j30w00hetbz.jpg)

### 段开销
 完成对STM-N整体信号流进行监控。即对STM-N“车厢”中所有“货物包”进行整体上的性能监控。其中，
 * 再生段开销(RSOH)—完成对STM-N整体信息结构进行监控
 * 复用段开销(MSOH)—完成对STM-N中的复用段层信息结构进行监控
 * RSOH、MSOH、POH组成SDH层层细化的监控体制
 * 二者区别：宏观（RSOH）和微观（MSOH）

 ### 管理单元指针——AU-PTR

 * 定位低速信号在STM-N帧中(净负荷)的位置，使低速信号在高速信号中的位置可预知。
 * 发端在将信号包装入STM-N净负荷时，加入AU-PTR，指示信号包在净负荷中的位置，即将装入“车厢”的“货物包”，赋予一个位置坐标值。
 * 收端根据AU指针值，从STM-N帧净负荷中直接拆分出所需的低速支路信号；即依据“货物包”位置坐标，从“车厢”中直接所需要的那一个“货包”。
 * 由于“车厢”中的“货物包”是以一定的规律摆放的——字节间插复用方式；所以对货物包的定位仅需定位“车厢”中第一个“货物包”即可。

 ![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x14ll91ej30s50gb0vy.jpg)


 若复用的低速信号速率较低，即打包后信息包太小，例：2M、34M。需进行二级指针定位。先将小信息包打包成中信息包，通过支路单元指针TU-PTR定位其在中信息包中的位置。然后将若干中信息包打包成大信息包，通过AU-PTR指示相应中信息包的位置。

 ![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x17bsdtxj30t60fwgmy.jpg)


## 复用步骤（复用方式，复用结构）
低阶SDH→高阶SDH：同步字节间插复用方式

PDH信号→STM-N：同步复用和灵活的映射
* 140M→STM-N
* 34M→STM-N
* 2M→STM-N


复用是依复用路线图进行的，ITU-T规定的路线图有多种，但通常一个国家或地区仅使用一种。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x1cus2p4j30t50k9783.jpg)

### 140M复用步骤

使用的是C4

C-4： 容器4；与140M相对应的标准信息结构，完成速率适配功能。

VC-4：虚容器4；与C-4相对应的标准信息结构，完成对
装载的140M信号进行实时的性能监控。

AU-4：管理单元4，与VC-4相对应的信息结构

复用路线140M—C-4—VC-4—AU-4—STM-1，所以STM-1
仅能复用进一路140M信号

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x1zj7lutj30rj099jt9.jpg)
![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x205i2s2j30n70a0tac.jpg)

从图中可以看到，在C-4容器中，是1-260，然后，加了一个POH，变成了1-261，这个就是之前介绍的净负荷区。
然后需要加一个指针定位AU-PTR，是一个1-9的，然后，加入段开销，就变成了一个完整的帧。

### 34M复用步骤

C-3： 容器3；与34M相对应的标准信息结构，完成速率适配功能。

VC-3： 虚容器3；与C-3相对应的标准信息结构，完成对装载的34M信号进行实时的性能监控。

TU-3： 支路单元3；与VC-3相对应的标准信息结构，完成一级指针
定位。

TUG-3： 支路单元组3；与TU-3相对应的标准信息结构。

复用路线34M—C-3—VC-3—TU-3—TUG-3；3TUG-3—VC-4—STM-
1；所以STM-1仅能复用进3路34M。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x31mfxvbj30pk08qwfx.jpg)
![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x328jauaj30q008k401.jpg)

 从图中可以看到，速度适配后，C-3是从1-84的，再加上一个POH，变成了1-85，由于不符合1-261，因此，可以将三个叠加起来，所以，先进行一级指针定位，然后，补齐缺口后，变成了1-86，这时候，将三个合在一起，$86\times3=258+3=261$,其中R是填充。这样就变成了一个VC-4，也就是一个STM-1。

 ### 2M复用步骤

C-12： 容器12；与2M相对应的标准信息结构，完成2M信号速率适
配，4个基帧组成一复帧。

VC-12：虚容器12；与2M相对应的标准信息结构，完成对某路2M
信号实时监控。

TU-12：支路单元12；与VC-12相对应的标准信息结构，完成对VC-
12的一级指针定位。

TUG-2：支路单元组2；TUG-3——支路单元组3。

2M—C-12—VC-12—TU-12；3TU-12—TUG-2；7TUG-2—TUG-
3；3TUG-3—VC-4—STM-1。

STM-1可装入3×7×3=63个2M信号。2M复用结构是3-7-3结构。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x3mviuwbj30qm07y0ul.jpg)
![](http://ww1.sinaimg.cn/large/006eDJDNly1g4x3nkjqlpj30oi07vq3u.jpg)

从图中，2M速度适配后变成了1-4，然后进行加POH监控和一级指针定位。之后进行3倍的字节间插，就变成了1-12，然后再进行7倍的字节间插，添加两个填充，就变成了1-86，之后再进行3倍字节间插（和34M是一样的），因此STM-1可以装$3\times7\times3=63$个2M信号。

