---
title: QoS
date: 2020-08-04 09:20:54
tags: ['数通','QoS']
categories: 数通
---
<meta name="referrer" content="no-referrer" />

# IP QoS三种模型
* Best-Effort模型：目前Internet的缺省服务模型，主要实现技术是先进先出队列（FIFO）
* IntServ模型: 业务通过信令向网络申请特定的QoS服务，网络在流量参数描述的范围内，预留资源以承诺满足该需求
* DiffServ模型：当网络出现拥塞时，根据业务的不同服务等级约定，有差别地进行流量控制和转发来解决拥塞问题

# FIFO模型
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghej7q1xlkj315c0llwtp.jpg)

# MQC配置模式

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghejgv4ao9j30zz0lpne1.jpg)

配置流程为：先配置class-map,再配置policy-map，再配置到service-map，然后再应用到端口即可

例子：
```

#先是分类，分等级

class-map match-all ipp7
   description for Diamond
   match precedence 7
end-class-map



# 然后对于每一个等级，进行带宽的分配，限速等配置
policy-map pm5-8000K-in
    class ipp7
      police rate 1200000 bps burst 150000 bytes peak-burst 150000 bytes
    
      conform-action set mpls experimental imposition7
      exceed-action drop
   ...


#对于总的带宽，进行一个配置

policy-map pm5-8000K-in.p
  class class-default
    service-policy pm5-8000K-in
    police rate 8000000 bps burst 1000000 bytes peak-burst 1000000 bytes
    conform-action transmit
    exceed-action drop



#把该policy应用到端口中去

interface GigabitEthernet0/0/0/3.142
  description For ...
  service-policy input pm5-8000K-in.p
  service-policy output pm5-8000K-out.p
  vrf VPN12432
  ipv4 address 10.122.122.122 255.255.255.252
  encapsulation dot1q 111


```


# VPN业务等级
* IPP7 = 钻石
* IPP5 = 白金
* IPP3+6 = 金
* IPP2 = 银
* IPP1 = 铜
* IPP0 = 其他


# 分类和标记（Classification and Marking）

对不同的包进行分类，如果没有分类的，则视为是一样的

分类完成后，需要对不同的包进行标记，通常使用IP包的IP precedence ，MPLS的EXP标记位进行分类


## 分类的配置
```
class-map match-all ipp7
   description for Diamond
   match precedence 7  # 匹配的是IP包的procedence标志位，也就是根据用户的IP包进行分类
end-class-map


class-map match-all ipp5
   description for Premium
   match precedence 5
end-class-map



class-map match-all ipp3
   description for Gold
   match precedence 3
end-class-map


class-map match-all ipp2
   description for Silver
   match precedence 2
end-class-map

```


## 标记配置

```
policy-map pm5-8000K-in
  class ipp7
    police rate 1200000 bps burst 150000 bytes peak-burst 150000 bytes
    conform-action set mpls experimental importion 7  #在MPLS的EXP上进行标记，逻辑上也就是从客户收过来IP包，通过IP Precedence进行分类，分类位ipp7的，在MPLS包上进行打标，设置为7
    exceed-action drop


  class ipp5
     police rate 1600000 bps burst 200000 bytes peak-burst 200000 bytes
    conform-action set mpls experimental importion 5
    exceed-action drop  


  class ipp3
    set mpls experimental importion 3 

  class ipp2
    set mpls experimental importion 2

  class class-default
    set mpls experimental importion 1

end-policy-map  
```

# 流量监管和整形


## 流量监管
流量监管是限制进入网络的流量与突发，为网络的稳定提供了基本的QoS功能。

流量监管TP（Traffic Policing）的典型应用是监督进入网络的某一流量的规格，把它限制在一个合理的范围之内，并对超出部分的流量进行“惩罚”，以保护网络资源和运营商的利益。



## 流量整形 

流量整形则是限制流出网络的流量与突发，为网络的稳定提供了基本的QoS功能。

流量整形TS（Traffic Shaping）的典型作用是限制流出某一网络的某一连接的正常流量与突发流量，使这类报文以比较均匀的速度向外发送，是一种主动调整流量输出速率的措施。



## Police限速

对于Policing，对于限制外的，就直接丢弃了

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghels3mlqpj30jl0lsqd4.jpg)


对于shaping，会进行一个平滑，对于超限的，会有一个缓存区先缓存着，缓存区满了才会进行丢弃

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghelurqctsj30jq0mvn6p.jpg)




```
policy-map pm5-8000K-in
  class ipp7
    #使用的是police配置
    #burst 允许突发一点，
    police rate 1200000 bps burst 150000 bytes peak-burst 150000 bytes
    conform-action set mpls experimental importion 7 
    exceed-action drop


  class ipp5
     police rate 1600000 bps burst 200000 bytes peak-burst 200000 bytes
    conform-action set mpls experimental importion 5
    exceed-action drop  
```






## Shape限速
```
policy-map pm5-8000K-out.p

  class class-default
  service-policy pm5-8000K-out
  shape average 8000000bps #shape限速

end-policy-map

```

# 拥塞管理及避免

* 网络拥塞时，保证不同的优先级的报文得到不同的QoS待遇
* 将不同优先级的报文入不同的队列，不同队列将得到不同的调度优先级，概率或带宽保证

## PQ: Priority Queuing

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghem6esxtjj311z0mc7hk.jpg)


问题：如果Queue 1一直不空的话，那么Queue 2 和Queue 3就饿死了 根本转发不了，因此这种策略往往只用在最重要的业务才会设置这个，优先转发

```
policy-map pm5-8000K-out
  class ipp7
    #配置一个PQ,
    priority level 1
    police rate percent 15
    conform-action transmit 
    exceed-action drop

```

## CBWFQ配置



* Priority：优先转发
* Bandwidth：带宽报障 在拥塞时发生作用，具体的就是，即使发生了拥塞，仍有这些带宽报障，当然，最高也不能超过Police
* Police：带宽限速

```
policy-map pm5-8000K-out
  class ipp7 
    priority level 1
    police rate percent 15
    conform-action transmit
    exceed-action drop


  class ipp5 
    bandwidth percent 5
    police rate percent 25
    conform-action transmit
    exceed-action drop

    queue-limit 32 packets

  class ipp3 
    bandwidth percent 30
    police rate percent 50
    conform-action transmit
    exceed-action drop

    queue-limit 384 packetsmit
    random-detect precedence 3 100 packets 280 packets



  class ipp2 
    bandwidth percent 25
    police rate percent 75
    conform-action transmit
    exceed-action drop

    queue-limit 384 packetsmit
    random-detect precedence 2 80 packets 225 packets



  class class-default 
    bandwidth percent 25
    queue-limit 384 packetsmit
    random-detect precedence 6,4,1,0,73 packets 150 packets

end-policy-map
```

## 拥塞避免

在网络没有发生拥塞以前根据队列状态进行有选择性的丢包

算法：
* RED  ： Random Early Detection
* WRED ： Weighted Random Early Detection

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghen0jj02xj31400nk12r.jpg)

横坐标时队列的大小，在0-32的时候，是不丢包的，在32-40的时候，是随机丢，40之后，就全部丢，tail drop == full drop

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghenccq7rxj311q0g77gf.jpg)

 ```
 # 针对优先级为3的流量使用WRED策略
 # 100以内，不丢，100-280 随机丢 
 random-detect precedence 3 100 packets 280 packets
 ```

# QoS相关命令
```
show running class-maps


show running policy-map


show running-config interface 


show policy-map interface  <interface> <input|output>


show qos interface <interface> <input|output>
```


* policed and dropped ：如果超过限速的值，那么就丢弃了
* RED random drops: 随机丢弃


![tempsnip.png](http://ww1.sinaimg.cn/large/006eDJDNly1gher5eljt9j319h0olqju.jpg)





# 华为的QoS



## In方向策略

通过上面思科系统的学习，知道了In方向通常只是对IP包进行分类（通过IP包的precedence），然后会有一个限速


### Classifier 分类

分类：根据IP包的precedence进行分类

```
traffic classifier dsipp7 operator or
  if-match ip-precedence 7

traffic classifier dsipp5 operator or
  if-match ip-precedence 5

traffic classifier dsipp3+6 operator or
  if-match ip-precedence 3
  if-match ip-precedence 6

traffic classifier dsipp2 operator or
  if-match ip-precedence 2

traffic classifier others operator or
  if-match ip-precedence 0
  if-match ip-precedence 1
  if-match ip-precedence 4 

```



### Behavior 动作

对不同类的一些行为操作，比如限速等

```


# cir表示指定承诺信息速率
# cbs表示指定突发尺寸，即令牌桶的容量
# pbs表示指定峰值突发尺寸

traffic behavior dsipp7
  car cir <D> cbs XXXX pbs XXXX
  service-class cs7 color green

traffic behavior dsipp5
  car cir <D+P> cbs XXXX pbs XXXX
  service-class ef color green

...


traffic behavior others
  service-class af1 color green

```


### policy

将classifier和behavior合并之后，就变成了policy，也就是对于每一个分类，都有相应的行为操作

Qos-profile的user-queue配置是对整个端口的限速，behavior中的限速是对某一个ipp的限速，Behavior的限速必定 小于等于 user-queue的限速

```
traffic policy XXX-in
  classifier dsipp7 behavior dsipp7 
  classifier dsipp5 behavior dsipp5
  classifier dsipp3+6 behavior dsipp3+6
  classifier dsipp2 behavior dsipp2
  classifier others behavior others


# cir表示指定承诺信息速率
# pir表示指定峰值速率

qos-profile XXX-in.p
  user-queue cir 2048 pir 2048

```


### 流量监管（CAR）

首先在接口下配置
```
interface Serial4/0/0/3:0
   traffic-policy AAA-2M-in inbound

```

其中，AAA-2M-in为
```
traffic policy AAA-2M-in
 share-mode
 classifier ipp7 behavior bh-ipp7-AAA-2M-in
 classifier ipp5 behavior bh-ipp5-AAA-2M-in
 classifier all behavior bh-ipp2-in

```
其中，bh-ipp7-AAA-2M-in
```
traffic behavior bh-ipp7-AAA-2M-in
 car cir 304 cbs 57000 pbs 57000 green pass yellow pass red discard
 service-class cs7 color green no-remark

```

其中，
```
traffic classifier ipp7 operator or
 if-match ip-precedence 7
 if-match mpls-exp 7

```

配置检查命令

查看behavior
```
display traffic behavior user-defined bh-ipp7-AAA-2M-in
```
查看traffic policy
```
display traffic policy user-defined AAA-2M-in
```
查看classifier
```
display traffic classifier user-defined ipp7
```


### 流量整形（flow-queue方式）


queue命令用来在流队列模板中修改流队列的调度参数，包括：调度方式、shaping、WRED。

参数：
* cos-value，指定配置的流队列，取值可以是af1～af4、be、cs6、cs7、ef
* weight-value，整数形式，取值范围是1～100
* shaping-value，整形速率，表示每个流队列的整形速率，整数形式，取值范围是8～4294967294，单位为Kbit/s。
* shaping-percentage-value，整形速率的百分比，表示整形带宽占Qos模板中用户队列峰值带宽的百分比。整数形式，取值范围是0～100
* pq | wfq | lpq ： 配置该队列的调度方式。pq为绝对优先级队列调度；wfq为加权公平队列调度；lpq为低优先级调度。三种队列调度的优先级次序为：PQ队列的优先级高于WFQ队列的优先级。WFQ队列的优先级高于LPQ队列的优先级。高优先级的队列可以抢占低优先级队列的带宽。
* pbs pbs-value，指定峰值突发尺寸（Peak Burst Size）
* wred-name，配置该队列使用的WRED对象


举例子：
```
在流队列WRED模板对象test中，配置af1队列的权重为50，整形速率为6Mbit/s。

queue af1 wfq weight 50 shaping 6000 pbs 1000 flow-wred test

```

#### 配置方法：

老的配置
```
#老的配置使用的是user-queue,新的使用的是qos-profile，
interface GigabitEthernet3/0/0.402
  user-queue cir 18227 pir 18227 flow-queue AAA-20M-out outbound


flow-queue AAA-20M-out
 queue af2 wfq weight 34 flow-wred ipp2-wred
 queue af3 wfq weight 66 flow-wred ipp3-wred
 queue cs7 pq shaping shaping-percentage 30

```

新的配置

```
interface GigabitEthernet2/1/0.153

 qos-profile AAA-6M-out.p outbound identifier none
 statistic enable


qos-profile AAA-6M-out.p
 user-queue cir 6144 pir 6144 flow-queue AAA-6M-out outbound


flow-queue AAA-6M-out
 queue af2 wfq weight 24 shaping shaping-percentage 100 flow-wred ipp2-wred
 queue af3 wfq weight 46 shaping shaping-percentage 76 flow-wred ipp3-wred
 queue ef wfq weight 30 shaping shaping-percentage 30 flow-wred ipp5-wred

```


查看配置命令：
```
 display flow-queue configuration verbose AAA-20M-out
```



### 拥塞管理

拥塞管理指网络在发生拥塞时，如何进行管理和控制。处理的方法是使用队列技术，将从一个接口发出的所有报文放入多个队列，按照各个队列的优先级进行处理。不同的队列调度算法用来解决不同的问题，并产生不同的效果。

比如，PQ队列，WFQ队列等等





### 拥塞避免
通过监视网络资源（如队列或内存缓冲区）的使用情况，在拥塞有加剧的趋势时，主动丢弃报文，通过调整网络的流量来解除网络过载的一种流量控制机制。拥塞避免用于防止因为线路拥塞而使设备的队列溢出。


```
interface GigabitEthernet6/0/2.100        
 
 qos-profile AAA-out.p outbound identifier none
 statistic enable



qos-profile AAA-out.p
 user-queue cir 2000000 pir 2000000 flow-queue AAA-out



flow-queue AAA-out
 queue cs7 pq flow-wred ipp3-wred


flow-wred ipp3-wred                       
 color green low-limit 50 high-limit 100 discard-percentage 100

```

