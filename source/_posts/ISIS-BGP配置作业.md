---
title: ISIS-BGP配置作业
date: 2020-02-14 11:15:03
tags: ['ISIS','BGP']
categories: 网络
---



# 网络拓扑图
![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gbvrbro5nfj317l0liagt.jpg)


# 配置

## 分配地址
在打开路由器的时候，默认的是用户视图，也就是命令行前面是<>，这时候如果要进入系统视图的话，就需要使用system-view,这样就进入到了系统视图，前面的命令行前面变成了[]
```
<Huawei> system-view
```

进入到每个端口：
```
[Huawei]interface GigabitEthernet 0/0/0
```

为每个端口配置ip地址
```
ip address + IP地址 + 子网掩码
[Huawei-GigabitEthernet0/0/0]ip address 192.168.1.100 255.255.255.0
```

## 配置ISIS
```
isis 1 # 配置ISIS进程，进程使用数字1
is-level level-2  #配置系统所属的区域为Level-2
cost-style wide   #配置ISIS开销值的类型为wide格式
network-entity 86.4134.0100.0000.0001.00 # 配置ISIS的NET值：86.AS号.loopback地址翻译.00
circuit-cost 100000 level-2  #配置ISIS路由的接口缺省开销值为100000
is-name PE1 #本地路由器上ISIS系统配置动态主机名
maximum load-balancing 32  #配置ISIS负载分担路径最大条数为32条

然后在端口下配置：
进入端口
interface + 接口号
isis enable 1 #开启isis进程
isis circuit-type p2p # 配置为p2p模式
isis circuit-level level-2
isis cost 1000 level-2 

```



## 配置BGP

CE1&CE2的AS号为65530

PE1~PE4的AS号为4809

CE3&CE4的AS号为65531

因为是通过loopback地址建立邻居，因此首先需要loopback地址可以ping通，使用isis协议建立loopback地址之间的连接即可

IBGP配置
```
bgp 4809  #定位路由器所在的AS号
router-id loopback地址     
peer 对端的loopback地址  as-number 对端的AS号
peer 对端的loopback地址  connect-interface LoopBack 0 #如果是使用loopback地址建立bgp邻居的话，则这一条命令是必须的
```

EBGP配置

首先EBGP路由器之间需要使用静态路由建立连接
```
ip route-static 对端的loopback地址 mask掩码 出去的端口  出去端口的对端ip地址
比如：
ip route-static 115.168.1.1 255.255.255.255 Ethernet 0/0/0 192.168.1.1
```
```
bgp 4809  #定位路由器所在的AS号
router-id loopback地址     
peer 对端的loopback地址  as-number 对端的AS号
peer 对端的loopback地址  connect-interface LoopBack 0 #如果是使用loopback地址建立bgp邻居的话，则这一条命令是必须的
```
最后要注意，要比IBGP多一个修改跳数，因此IBGP默认跳数是255，但是EBGP默认跳数是1，需要改成大于1的数
```
peer 对端的loopback地址 ebgp-max-hop 2
```