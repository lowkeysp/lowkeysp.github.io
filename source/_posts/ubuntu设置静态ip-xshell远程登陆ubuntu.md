---
title: ubuntu设置静态ip-xshell远程登陆ubuntu
date: 2021-06-27 16:02:37
tags:
---
<meta name="referrer" content="no-referrer" />

# 设置静态IP地址

在`/etc/netplan/`目录下，有一个yaml文件
```
lowkeysp-00@lowkeysp-00:/etc/netplan$ ls
01-network-manager-all.yaml

```

对该文件进行编辑，修改
```shell
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:            #通过ifconfig查看网卡
      dhcp4: no        # 不适用DHCP，使用静态
      addresses: [192.168.10.1/24]    # IP地址/掩码
      gateway4: 192.168.10.2      # 网关地址
      nameservers: 
          addresses: [192.168.10.2]    # DNS地址

```

编辑完成后，
```shell
sudo netplan apply
```
之后，可以通过`ifconfig`查看确实已经更改为静态地址
```shell
lowkeysp-00@lowkeysp-00:/etc/netplan$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.1  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fef8:63de  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:f8:63:de  txqueuelen 1000  (以太网)
        RX packets 552  bytes 53961 (53.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 515  bytes 66516 (66.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (本地环回)
        RX packets 182  bytes 14978 (14.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 182  bytes 14978 (14.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```


# 修改VMWare的地址
【编辑】-【虚拟网络编辑器】

选中 VMnet8 那一行，点击右下角的【更改设置】

之后，将下方的子网IP改为：192.168.10.0，再点击【NAT设置】，网关IP改为：192.168.10.2，点击确定，再点击应用

# 修改Win系统地址

【网络和Internet设置】-【更改适配器选项】

选择【VMWare Network Adapter VMNet8】，右键-【属性】，【Internet协议版本4】，点击属性，修改IP地址为：192.168.10.101，默认网关是192.168.10.2，DNS服务器为：8.8.8.8，点击确定


# 在ubuntu上装SSH服务器，用于SSH登陆
```
sudo apt-get install openssh-server
```

# 使用xshell，用ssh方式即可登陆




