---
title: ubuntu下安装anaconda
date: 2019-07-22 14:17:42
tags: [ubuntu,anaconda,安装,配置]
categories: 安装
---


## 参考文献


https://askubuntu.com/questions/505919/how-to-install-anaconda-on-ubuntu

https://www.cnblogs.com/devilmaycry812839668/p/10349602.html


##  具体过程

这里我是给我的腾讯云服务器上面装的miniconda。
首先，使用清华镜下载Miniconda3-latest-Linux-x86_64.sh
```
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

下载好之后，执行
```
bash Miniconda3-latest-Linux-x86_64.sh
```

这样就安装好了，如果你发现你的python3不是最新的话，需要进行环境变量配置，在.bashrc文件里添加
```
export PATH="/home/ubuntu/miniconda3/bin:$PATH"
```

还有一个问题就是配置好之后，在命令行前面会出现一个base，去除base的原因是第二个参考文献给出，解决方法如下：
```
conda deactivate
```


