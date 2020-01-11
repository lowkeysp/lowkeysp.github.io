---
title: pytorch的Tensors学习
date: 2019-12-12 21:37:05
tags: ['pytorch','Tensors']
categories: pytorch
---


# 参考链接
http://pytorch123.com/SecondSection/what_is_pytorch/


# Tensors
Tensors 类似于 NumPy 的 ndarrays ，同时 Tensors 可以使用 GPU 进行计算。

比如，我们构造一个5*3的矩阵，不初始化

```
>>> import torch
>>> x = torch.empty(5,3)
>>> x
tensor([[-1.2141e-25,  4.5783e-41, -1.2141e-25],
        [ 4.5783e-41,  1.3563e-19,  1.3563e-19],
        [ 1.3563e-19,  1.3563e-19,  1.3563e-19],
        [ 1.3563e-19,  6.1678e+16,  6.4890e-07],
        [ 2.6176e-12,  4.0058e-11,  2.5787e-09]])

```

构造一个随机初始化的矩阵：
```
>>> torch.rand(5,3)
tensor([[0.0276, 0.6181, 0.1152],
        [0.2565, 0.2421, 0.5149],
        [0.9311, 0.2453, 0.3041],
        [0.6361, 0.8637, 0.1596],
        [0.7435, 0.8903, 0.9017]])
```
构造一个矩阵为全0，而且数据类型是long
```
>>> torch.zeros(5,3,dtype=torch.long)
tensor([[0, 0, 0],
        [0, 0, 0],
        [0, 0, 0],
        [0, 0, 0],
        [0, 0, 0]])
```
构造一个张量，直接使用数据：
```
>>> torch.tensor([5.5,3])
tensor([5.5000, 3.0000])
```
创建一个**基于一个已知的tensor**的tensor
```
>>> x = x.new_ones(5,3,dtype = torch.double)
>>> x
tensor([[1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.]], dtype=torch.float64)

#创建一个和x类似的y
>>> y = torch.randn_like(x,dtype=torch.float)
>>> y
tensor([[ 0.0115,  0.2386,  0.6542],
        [-0.1891, -0.6507,  1.0276],
        [ 1.2823, -0.1774,  0.1024],
        [-0.0743,  0.7157, -1.0440],
        [-0.7562, -0.6935, -0.0515]])
```

获得tensor的维度信息
```
>>> x.size()
torch.Size([5, 3])
```

加法操作

第一种：
```
x+y
```
第二种
```
torch.add(x,y)

#使用下面方法，可以将结果付给result
torch.add(x,y,out=result) 
```
第三种
```
y.add_(x)

# 解释： 把x加到y上去，类似于 y = x + y
```

可以使用标准的Numpy类似的索引操作
```
x[:,1]
```
如果你想改变一个tensor的大小或者形状，可以使用torch.view
```
>>> x = torch.randn(4,4)
>>> y = x.view(16)
>>> z = x.view(-1,8) #-1表示是另一个维度，即列向量
>>> x.size()
torch.Size([4, 4])
>>> y.size()
torch.Size([16])
>>> z.size()
torch.Size([2, 8])
```

如果你有一个元素tensor，可以使用.item()来获得这个value，
```
>>> x = torch.randn(1)
>>> x
tensor([-0.2879])
>>> x.item()
-0.28786054253578186
``` 

