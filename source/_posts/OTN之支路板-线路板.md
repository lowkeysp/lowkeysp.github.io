---
title: OTN之支路板&线路板
date: 2019-08-15 16:12:55
tags: [OTN,支路板,线路板]
categories: OTN
---

<meta name="referrer" content="no-referrer" />

华为OTN产品系列支持 支路板、线路板分离架构 。支路/线路板和集中交叉单板配合使用，除了可以完成OTU单板功能外，还可通过集中交叉单板进行各级别ODUk颗粒业务调度， 实现更加灵活的电层信号调度及更高的带宽利用率。


# 支路板

## 功能

实现客户侧业务在本站上/下波分侧。

## 在系统中的位置

![](http://ww1.sinaimg.cn/large/006eDJDNly1g60fyh0b5pj30mg0d8tas.jpg)



# 线路板

## 功能

* 配合支路板完成本站客户侧业务上/下波分侧

  * OTN支路板：接入本站的客户侧业务，完成O/E转换后，将客户侧业务映射为ODUk信号，并将ODUk信号发送到交叉板进行调度。
  * OTN线路板：将调度过来的ODUk信号进行复用和映射，完成E/O转换后， 转换为符合WDM系统要求的标准波长的OTUk信号。同时实现上述过程的逆过程。

* 配合OTN线路板完成波分侧业务的本站穿通
  * OTN支路板：接收西向的波分侧业务，完成光/电转换后，将波分侧业务映射和 解复用为ODUk信号，并将ODUk信号发送到交叉板进行调度。 
  * OTN线路板：将调度过来的ODUk信号进行复用和映射，完成电/光转换后， 转换为符合WDM系统要求的标准波长的OTUk信号，传输到东向。 同时实现上述过程的逆过程。

## 在系统中的位置

![](http://ww1.sinaimg.cn/large/006eDJDNly1g60gjt0gljj30lh0d8go0.jpg)


# 为什么需要采用支线路分离架构？

利用OTU单板即可以完成客户侧业务上波分侧，为什么还需要采用支线路分离的架构？

![](http://ww1.sinaimg.cn/large/006eDJDNly1g60gljxh8hj312d0bajtq.jpg)

可以看到，支线路分离架构与OTU架构的主要区别在于引入了集中交叉板。客户侧业务不是直接通过一块单板封装为波分侧业务（OTUk)上线路侧，而是通过集中交叉板进行 各级别的ODUk颗粒的调度。即增加了电层调度的灵活性，提高了带宽的利用率。

# 参考文献
http://support.huawei.com/onlinetoolweb/ptmngsys/Web/WDMkg/zh/25_zhiluban.html
