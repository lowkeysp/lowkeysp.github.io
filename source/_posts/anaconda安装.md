---
title: anaconda安装
date: 2019-12-11 23:23:29
tags: [anaconda,安装]
categories: 安装
---

<meta name="referrer" content="no-referrer" />

看到了一篇文章，感觉讲的挺好的，mark下来
https://www.jianshu.com/p/eaee1fadc1e9


另外，使用conda安装pytorch的时候，下载速度十分的慢，解决方法也很简单,就是将下载源更换为清华源即可，具体做法如下：

```
conda config   -- add  channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config   --add   channels  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config   --set   show_channel_urls  yes
```
同时添加第三方的下载源，（强烈建议添加，否则有可能在清华源找不到这个库）
```
conda config     --add    channels   https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config     --add    channels   https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config     --add    channels   https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/

conda config     --add    channels   https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/

conda config     --add    channels   https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
```

根据Python和CUDA选择对应的版本，然后官方给出提示可通过运行：

conda install pytorch torchvision cudatoolkit=9.0 -c pytorch

但是这里一定要注意，去掉-c pytorch，安装的时候才会默认从清华源下载相应的包，