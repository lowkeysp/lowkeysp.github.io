---
title: QoS
date: 2020-08-04 09:20:54
tags: ['数通','QoS']
categories: 数通
---

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


对于Policing，对于限制外的，就直接丢弃了

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghels3mlqpj30jl0lsqd4.jpg)


对于shaping，会进行一个平滑，对于超限的，会有一个缓存区先缓存着，缓存区满了才会进行丢弃

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1ghelurqctsj30jq0mvn6p.jpg)


## Police限速

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