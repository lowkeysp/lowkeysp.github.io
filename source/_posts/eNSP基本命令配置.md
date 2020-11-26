---
title: eNSP基本命令配置
date: 2019-05-23 14:53:10
tags: [eNSP,网络]
categories: 网络
---
<meta name="referrer" content="no-referrer" />

## 常用命令

### System-View
在打开路由器的时候，默认的是用户视图，也就是命令行前面是<>，这时候如果要进入系统视图的话，就需要使用system-view,这样就进入到了系统视图，前面的命令行前面变成了[]

```
<Huawei> system-view
```

### interface + 接口名
这是用来配置接口的，使用这个命令之后，我们就进入到该接口可以进行配置。比如
```
[Huawei]interface GigabitEthernet 0/0/0
```
这样就进入到了该接口，显示为
```
[Huawei-GigabitEthernet0/0/0]
```

### ip address + IP地址 + 子网掩码
用来给接口配置IP地址的，前提是需要进入到该接口
```
[Huawei-GigabitEthernet0/0/0]ip address 192.168.1.100 255.255.255.0
```



