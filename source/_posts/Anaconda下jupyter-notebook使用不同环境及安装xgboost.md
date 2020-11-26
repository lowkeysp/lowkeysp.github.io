---
title: Anaconda下jupyter notebook使用不同环境及安装xgboost
date: 2020-01-11 15:39:03
tags: ['Anaconda','jupyter notebook','xgboost']
categories: Anaconda
---

<meta name="referrer" content="no-referrer" />

# Anaconda下jupyter notebook使用不同环境
Anaconda的出世就是为了解决管理不同的环境配置的。具体的可以看我之前的一篇博客。而在Anaconda中，经常需要使用jupyter notebook，因此，希望可以在jupyter notebook中切换环境，具体做法如下：

第一步

安装ipykernel
```
conda install ipykernel
```

第二步

将conda环境写入jupyter notebook中，（conda环境的创建可以看之前的博客）。

```
python -m ipykernel install --user --name 环境名称 --display-name "你希望看见的环境名"
```

晚上上面两步之后，就可以在jupyter notebook中的kernel-change kernel中选择不同的环境了

如果想删除kernel环境的话，可以使用
```
jupyter kernelspec remove 环境名称
```

在创建conda环境时也可以同时创建kernel，如
```
conda create -n 环境名称  pyhont=3.*/2.* ipykernel
```

 参考：https://blog.csdn.net/Mr_liuxy/article/details/100011867
 



# 安装xgboost的GPU加速版

在安装xgboost的时候，如果只是安装CPU版本的话，就相当简单了，就一条命令，
```
pip install xgboost
```

但是如果你想安装GPU版本的话，一条命令是完成不了的，需要踩好多坑，现就将我成功的方式总结如下

这次我使用的是win10+anaconda

第一步

下载 xgboost 源码：https://github.com/dmlc/xgboost

第二步

从 http://ssl.picnet.com.au/xgboost/ 中下载支持GPU版已编译好的DLL文件

第三步

将xgboost.dll复制到xgboost-master/python-package/xgboost目录下，打开控制台，进入到python-package目录下，执行python setup.py install命令，然后，需要把xgboost-master文件名改成xgboost。至此xgboost GPU版安装成功

这时候，你以为你成功了，其实并没有，会报各种错误。过了两天，我忘记了有些错误的解决方法，这里就先提供还记得的方法，以后想起来了再补充

## 运行jupyter 报raise NotImplementedError，
解决方法：

找到：C:\Users\xxx\AppData\Local\Programs\Python\Python38\Lib\site-packages\tornado\platform\asyncio.py 这个文件，

打开将下边这句编辑进去，保存。

```
import asyncio
import sys
if sys.platform == 'win32' and sys.version_info > (3, 8, 0, 'alpha', 3) :
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```





## 检测xgboost GPU版本是否安装成功

```
import xgboost as xgb
import numpy as np
from sklearn.datasets import fetch_covtype
from sklearn.model_selection import train_test_split
import time

# Fetch dataset using sklearn
cov = fetch_covtype()
X = cov.data
y = cov.target

# Create 0.75/0.25 train/test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, train_size=0.75, random_state=42)

# Specify sufficient boosting iterations to reach a minimum
num_round = 25 #3000

# Leave most parameters as default
param = {'objective': 'multi:softmax', # Specify multiclass classification
         'num_class': 8, # Number of possible output classes
         'tree_method': 'gpu_hist' # Use GPU accelerated algorithm
         }


# Convert input data from numpy to XGBoost format
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)

gpu_res = {} # Store accuracy result
tmp = time.time()
# Train model
param['tree_method'] = 'gpu_hist'
xgb.train(param, dtrain, num_round, evals=[(dtest, 'test')], evals_result=gpu_res)
print("GPU Training Time: %s seconds" % (str(time.time() - tmp)))



# Repeat for CPU algorithm
tmp = time.time()
param['tree_method'] = 'hist'
cpu_res = {}
xgb.train(param, dtrain, num_round, evals=[(dtest, 'test')], evals_result=cpu_res)
print("CPU Training Time: %s seconds" % (str(time.time() - tmp)))
```


参考：

https://blog.csdn.net/weixin_30963287/article/details/79145107

https://zhuanlan.zhihu.com/p/89888038

https://www.cnblogs.com/chenxiangzhen/p/10774000.html
