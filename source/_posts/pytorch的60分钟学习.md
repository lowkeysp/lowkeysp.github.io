---
title: pytorch的60分钟学习
date: 2020-6-29 21:37:05
tags: ['pytorch','Tensors']
categories: pytorch
mathjax: true
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

Torch Tensor 和 Numpy Array 是使用的同样的内存位置（前提是Torch Tensor 是在CPU上运行的），更改其中的一个也会影响到另一个

把Torch Tensor 转换成 Numpy Array
```
>>a = torch.ones(5)
>>print(a)

out:
tensor([1.,1.,1.,1.,1.,])


>> b = a.numpy()
>> print(b)

out:
[1. 1. 1. 1. 1.]
```
让我们看看改变其中一个，另一个是否也会改变吧
```
>> a.add_(1)
>> print(a)
>> print(b)

out:
tensor([2., 2., 2., 2., 2.])
[2. 2. 2. 2. 2.]
```
看到没，我们只是改变了a，但是b也相应发生了改变，因为他们是共享同一块内存的

把Numpy Array转换成Torch Tensor
```
import numpy as np
a = np.ones(5)
b = torch.from_numpy(a)
np.add(a, 1, out=a)
print(a)
print(b)


out:
[2. 2. 2. 2. 2.]
tensor([2., 2., 2., 2., 2.], dtype=torch.float64)
```
也是，改变Numpy Array也会改变torch tensor

在CPU上，除了CharTensor之外，其他所有的Tensor都可以转换成Numpy Array



可以使用`.to`方法让Tensor支持任意设备
```
# let us run this cell only if CUDA is available
# We will use ``torch.device`` objects to move tensors in and out of GPU
if torch.cuda.is_available():
    device = torch.device("cuda")          # a CUDA device object
    y = torch.ones_like(x, device=device)  # directly create a tensor on GPU
    x = x.to(device)                       # or just use strings ``.to("cuda")``
    z = x + y
    print(z)
    print(z.to("cpu", torch.double))       # ``.to`` can also change dtype together!


out:
tensor([0.6469], device='cuda:0')
tensor([0.6469], dtype=torch.float64)
```
可以看到，第一个tensor是cuda的，第二个就是正常的。



# AUTOGRAD: AUTOMATIC DIFFERENTIATION

`autograd`这个包可以提供自动梯度。

'torch.Tensor'是这个包的核心类，如果你设置Tensor的'.requires_grad'为'True'的话，那么它会开始追踪你的所有的运算，进而可以算梯度。当你完成计算时，只需要调用'.backward()',就可以自动地计算出所有的梯度。这个tensor的梯度会被累加在这个'.grad'属性值里

使用'.detach()'可以停止追踪计算

还可以使用代码块'with torch.no_grad()'来停止追踪计算。这在评估模型的时候会非常有用，因为评估模型的时候，模型可能会含有可训练的参数，且'.requires_grad = True'，但是在评估模型的时候，并不需要梯度，因此可以使用这个代码块关闭追踪计算

'Function'这个类在计算梯度的时候也是非常重要的。

每一个Tensor都有一个'.grad_fn'属性，用来指向创建这个Tensor的'Function'，除非这个Tensor在创建的时候，人为的设置'grad_fn'属性为'None'（下面的例子有解释）


举例子：
```
import torch

x = torch.ones(2, 2, requires_grad=True)
print(x)


out:
tensor([[1., 1.],
        [1., 1.]], requires_grad=True)
```

对这个x进行一次加法操作
```
y = x + 2
print(y)


out:

tensor([[3., 3.],
        [3., 3.]], grad_fn=<AddBackward0>)
```


这个y是通过加法运算得来的，因此他会有'grad_fn'属性
```
print(y.grad_fn)

out:
<AddBackward0 object at 0x7f191afd60f0>
```

对y再进行运算
```
z = y * y * 3
out = z.mean()

print(z, out)


out:
tensor([[27., 27.],
        [27., 27.]], grad_fn=<MulBackward0>) tensor(27., grad_fn=<MeanBackward0>)
```

`.requires_grad_( ... )`可以改变Tensor的`requires_grad`属性。如果不指定的话，默认是False的。下面这个例子，刚开始a的`requires_grad`属性默认是False的，后来通过这个函数，可以改成True。

```
a = torch.randn(2, 2)
a = ((a * 3) / (a - 1))
print(a.requires_grad)
a.requires_grad_(True)
print(a.requires_grad)
b = (a * a).sum()
print(b.grad_fn)



out：
False
True
<SumBackward0 object at 0x7f191afd6be0>
```

现在来计算Backprop,因为out是一个标量，所以`out.backward()`相当于是`out.backward(torch.tensor(1.))`

```
out.backward()
```

打印d(out)/dx 的梯度
```
print(x.grad)

out:
tensor([[4.5000, 4.5000],
        [4.5000, 4.5000]])
```

这个4.5的值是怎么得到的呢？

Tensor `out` 表示成$o$,我们有$o = \frac{1}{4}\sum_i z_i$,$z_i = 3(x_i+2)^2$,和$z_i\bigr\rvert_{x_i=1} = 27$。因此，有$\frac{\partial o}{\partial x_i} = \frac{3}{2}(x_i+2)$,所以，$\frac{\partial o}{\partial x_i}\bigr\rvert_{x_i=1} = \frac{9}{2} = 4.5$

在数学上，如果你有一个关于向量的函数，$\vec{y}=f(\vec{x})$,那么关于$\vec{x}$的$\vec{y}$的梯度是一个Jacobian矩阵:

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gg8xlresm2j307f04gmx1.jpg)

因此，可以说`torch.autograd`是用来计算vector-Jacobian product。也就是，给任意一个向量，$v=\left(\begin{array}{cccc} v_{1} & v_{2} & \cdots & v_{m}\end{array}\right)^{T}$,计算$v^{T}\cdot J$. 如果$v$刚好是函数$l=g\left(\vec{y}\right)$的梯度，也就是$v=\left(\begin{array}{ccc}\frac{\partial l}{\partial y_{1}} & \cdots & \frac{\partial l}{\partial y_{m}}\end{array}\right)^{T}$,那么根据chain rule，$v^{T}\cdot J$就是我们要求的$l$关于$\vec{x}$的梯度。（总结一下，就是如果要求梯度的话，如果我们有了Jacobian矩阵和向量$v$，那么我们就可以求出这个梯度）

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gg8xuj85rjj30bl04v3yi.jpg)


我们来看一个例子
```
x = torch.randn(3, requires_grad=True)

y = x * 2
while y.data.norm() < 1000:
    y = y * 2

print(y)


out:
tensor([-970.9141,  465.0858, 1599.4425], grad_fn=<MulBackward0>)
```

在这个例子里，$y$不是一个scalar，'torch.autograd'无法直接计算整个Jocobian，但是，如果我们只想要vector-Jacobian product的话，只需要把一个向量传给`backward`即可

```
v = torch.tensor([0.1, 1.0, 0.0001], dtype=torch.float)
y.backward(v)

print(x.grad)



out:
tensor([1.0240e+02, 1.0240e+03, 1.0240e-01])
```


当想要停止追踪计算时，如果Tensor的`requires_grad`属性是True时，可以使用`with torch.no_grad():`
```
print(x.requires_grad)
print((x ** 2).requires_grad)

with torch.no_grad():
    print((x ** 2).requires_grad)


out:
True
True
False
```

还可以使用`.detach()`得到一个新的Tensor，这个Tensor和之前的Tensor是一样的，只是没有了`requires_grad=True`
```
print(x.requires_grad)
y = x.detach()
print(y.requires_grad)
print(x.eq(y).all())


out:
True
False
tensor(True)
```

参考一篇知乎讲解：https://zhuanlan.zhihu.com/p/65609544


# NEURAL NETWORKS

神经网络的搭建可以使用`torch.nn`包

一个'nn.Module'包括layers，一个`forward(input)`方法和一个`output`


一个神经网络的训练过程如下：

* Define the neural network that has some learnable parameters (or weights)
* Iterate over a dataset of inputs
* Process input through the network
* Compute the loss (how far is the output from being correct)
* Propagate gradients back into the network’s parameters
* Update the weights of the network, typically using a simple update rule: `weight = weight - learning_rate * gradient`


![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gg8ztjlr3pj315f0cn77x.jpg)

让我们定义这个神经网络：

```
import torch
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):

    def __init__(self):
        super(Net, self).__init__()
        # 1 input image channel, 6 output channels, 3x3 square convolution
        # kernel
        self.conv1 = nn.Conv2d(1, 6, 3)
        self.conv2 = nn.Conv2d(6, 16, 3)
        # an affine operation: y = Wx + b
        self.fc1 = nn.Linear(16 * 6 * 6, 120)  # 6*6 from image dimension
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        # Max pooling over a (2, 2) window
        x = F.max_pool2d(F.relu(self.conv1(x)), (2, 2))
        # If the size is a square you can only specify a single number
        x = F.max_pool2d(F.relu(self.conv2(x)), 2)
        x = x.view(-1, self.num_flat_features(x))
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

    def num_flat_features(self, x):
        size = x.size()[1:]  # all dimensions except the batch dimension
        num_features = 1
        for s in size:
            num_features *= s
        return num_features


net = Net()
print(net)







out:

Net(
  (conv1): Conv2d(1, 6, kernel_size=(3, 3), stride=(1, 1))
  (conv2): Conv2d(6, 16, kernel_size=(3, 3), stride=(1, 1))
  (fc1): Linear(in_features=576, out_features=120, bias=True)
  (fc2): Linear(in_features=120, out_features=84, bias=True)
  (fc3): Linear(in_features=84, out_features=10, bias=True)
)
```


你只需要定义`forward`函数,  `backward`函数会自动定义的（使用`autograd`）。你可以在`forward`函数里使用任意Tensor的operation


模型里面的可学习的参数可以通过`net.parameters()`学到
```
params = list(net.parameters())
print(len(params))
print(params[0].size())  # conv1's .weight


out:
10
torch.Size([6, 1, 3, 3])
```

让我们试一下32*32的input。
```
input = torch.randn(1, 1, 32, 32)
out = net(input)
print(out)


out:
tensor([[ 0.1242,  0.1194, -0.0584, -0.1140,  0.0661,  0.0191, -0.0966,  0.0480,
          0.0775, -0.0451]], grad_fn=<AddmmBackward>)
```

之后清零 所有参数的梯度缓存 然后 使用backprops
```
net.zero_grad()
out.backward(torch.randn(1, 10))
```

* NOTE:
>torch.nn only supports mini-batches. The entire torch.nn package only supports inputs that are a mini-batch of samples, and not a single sample.
>
>For example, nn.Conv2d will take in a 4D Tensor of nSamples x nChannels x Height x Width.
>
>If you have a single sample, just use input.unsqueeze(0) to add a fake batch dimension.


回顾一下学到的：

* `torch.Tensor` - A multi-dimensional array with support for autograd operations like `backward()`. Also holds the gradient w.r.t. the tensor.
* `nn.Module` - Neural network module. Convenient way of encapsulating parameters, with helpers for moving them to GPU, exporting, loading, etc.
* `nn.Parameter` - A kind of Tensor, that is automatically registered as a parameter when assigned as an attribute to a `Module`.
* `autograd.Function` - Implements forward and backward definitions of an autograd operation. Every `Tensor` operation creates at least a single `Function` node that connects to functions that created a `Tensor` and encodes its history.


损失函数：

```
output = net(input)
target = torch.randn(10)  # a dummy target, for example
target = target.view(1, -1)  # make it the same shape as output
criterion = nn.MSELoss()

loss = criterion(output, target)
print(loss)


out:
tensor(1.1562, grad_fn=<MseLossBackward>)
```

这时候，使用'.grad_fn'，可以看到一个计算的流程图：
```
input -> conv2d -> relu -> maxpool2d -> conv2d -> relu -> maxpool2d
      -> view -> linear -> relu -> linear -> relu -> linear
      -> MSELoss
      -> loss
```

因此，当我们调用`loss.backward()`时，则会计算梯度

为了说明backward流程，看下面的例子：
```
print(loss.grad_fn)  # MSELoss
print(loss.grad_fn.next_functions[0][0])  # Linear
print(loss.grad_fn.next_functions[0][0].next_functions[0][0])  # ReLU


out:
<MseLossBackward object at 0x7fdba3216da0>
<AddmmBackward object at 0x7fdba3216f28>
<AccumulateGrad object at 0x7fdba3216f28>
```

Backprop:

反向传播使用`loss.backward()`即可，在这之前，需要清除之前存在的梯度，否则现有的梯度计算会累加到之前的梯度种
```
net.zero_grad()     # zeroes the gradient buffers of all parameters

print('conv1.bias.grad before backward')
print(net.conv1.bias.grad)

loss.backward()

print('conv1.bias.grad after backward')
print(net.conv1.bias.grad)


out:
conv1.bias.grad before backward
tensor([0., 0., 0., 0., 0., 0.])
conv1.bias.grad after backward
tensor([-0.0002,  0.0045,  0.0017, -0.0099,  0.0092, -0.0044])
```

更新weight：

pytorch提供了各种更新规则（比如：SGD，Nesterov—SGD，Adam，RMSProp，等）的包`torch.optim`。使用起来也非常建单：
```

# create your optimizer
optimizer = optim.SGD(net.parameters(), lr=0.01)

# in your training loop:
optimizer.zero_grad()   # zero the gradient buffers
output = net(input)
loss = criterion(output, target)
loss.backward()
optimizer.step()    # Does the update
```



# TRAINING A CLASSIFIER

一般地，当你处理数据的时候，比如图像，文本，语音，或者视频，你可以使用标准的python库加载数据，编程numpy array，然后转换numpy array 到 `torch.*Tensor`
* For images, packages such as Pillow, OpenCV are useful
* For audio, packages such as scipy and librosa
* For text, either raw Python or Cython based loading, or NLTK and SpaCy are useful

特别是对于vision，pytorch有一个包叫`torchvision`，可以便捷地加载通用的数据集，比如 Imagenet, CIFAR10, MNIST，

我们将使用CIFAR10的数据集，有‘airplane’, ‘automobile’, ‘bird’, ‘cat’, ‘deer’, ‘dog’, ‘frog’, ‘horse’, ‘ship’, ‘truck’. 这几类，图片的大小是3*32*32，其中3-channel color image of 32*32 pixels in size

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gg9bpvyihwj30rj0jl4qp.jpg)

为了训练一个图片分类器，我们需要做一下几点：
* Load and normalizing the CIFAR10 training and test datasets using torchvision
* Define a Convolutional Neural Network
* Define a loss function
* Train the network on the training data
* Test the network on the test data


## Loading and normalizing CIFAR10


The output of torchvision datasets are PILImage images of range [0, 1]. We transform them to Tensors of normalized range [-1, 1]. .. note:
>If running on Windows and you get a BrokenPipeError, try setting the num_worker of torch.utils.data.DataLoader() to 0.


```
import torch
import torchvision
import torchvision.transforms as transforms


transform = transforms.Compose(
    [transforms.ToTensor(),
     transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))])

trainset = torchvision.datasets.CIFAR10(root='./data', train=True,
                                        download=True, transform=transform)
trainloader = torch.utils.data.DataLoader(trainset, batch_size=4,
                                          shuffle=True, num_workers=2)

testset = torchvision.datasets.CIFAR10(root='./data', train=False,
                                       download=True, transform=transform)
testloader = torch.utils.data.DataLoader(testset, batch_size=4,
                                         shuffle=False, num_workers=2)

classes = ('plane', 'car', 'bird', 'cat',
           'deer', 'dog', 'frog', 'horse', 'ship', 'truck')





out：
Extracting ./data/cifar-10-python.tar.gz to ./data
Files already downloaded and verified
```

下面的代表可以用来展示训练的图片
```
import matplotlib.pyplot as plt
import numpy as np

# functions to show an image


def imshow(img):
    img = img / 2 + 0.5     # unnormalize
    npimg = img.numpy()
    plt.imshow(np.transpose(npimg, (1, 2, 0)))
    plt.show()


# get some random training images
dataiter = iter(trainloader)
images, labels = dataiter.next()

# show images
imshow(torchvision.utils.make_grid(images))
# print labels
print(' '.join('%5s' % classes[labels[j]] for j in range(4)))
```



## Define a Convolutional Neural Network

```
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.pool = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x


net = Net()
```

## Define a Loss function and optimizer
使用的是 Cross-Entropy loss 和 SGD with momentum

```
import torch.optim as optim

criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)


```


## Train the network

我们只需要循环我们的 data iterator，然后把input喂给网络即可

```
for epoch in range(2):  # loop over the dataset multiple times

    running_loss = 0.0
    for i, data in enumerate(trainloader, 0):
        # get the inputs; data is a list of [inputs, labels]
        inputs, labels = data

        # zero the parameter gradients
        optimizer.zero_grad()

        # forward + backward + optimize
        outputs = net(inputs)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

        # print statistics
        running_loss += loss.item()
        if i % 2000 == 1999:    # print every 2000 mini-batches
            print('[%d, %5d] loss: %.3f' %
                  (epoch + 1, i + 1, running_loss / 2000))
            running_loss = 0.0

print('Finished Training')




out:
[1,  2000] loss: 2.227
[1,  4000] loss: 1.884
[1,  6000] loss: 1.672
[1,  8000] loss: 1.582
[1, 10000] loss: 1.526
[1, 12000] loss: 1.474
[2,  2000] loss: 1.407
[2,  4000] loss: 1.384
[2,  6000] loss: 1.362
[2,  8000] loss: 1.341
[2, 10000] loss: 1.331
[2, 12000] loss: 1.291
Finished Training
```

保存我们的模型：
```
PATH = './cifar_net.pth'
torch.save(net.state_dict(), PATH)
```

## Test the network on the test data


我们可以先拿出几个测试集的样本 看看预测的怎么样
```
dataiter = iter(testloader)
images, labels = dataiter.next()

# print images
imshow(torchvision.utils.make_grid(images))
print('GroundTruth: ', ' '.join('%5s' % classes[labels[j]] for j in range(4)))


#加载训练好的模型
net = Net()
net.load_state_dict(torch.load(PATH))


#来看看模型预测的结果是什么
outputs = net(images)

_, predicted = torch.max(outputs, 1)

print('Predicted: ', ' '.join('%5s' % classes[predicted[j]]
                              for j in range(4)))

```

我们可以看看在整个测试集上面训练的结果如何
```
correct = 0
total = 0
with torch.no_grad():
    for data in testloader:
        images, labels = data
        outputs = net(images)
        _, predicted = torch.max(outputs.data, 1)
        total += labels.size(0)
        correct += (predicted == labels).sum().item()

print('Accuracy of the network on the 10000 test images: %d %%' % (
    100 * correct / total))
```

我们还可以看看哪些类模型识别的比较好，哪些类模型识别的比较差
```
class_correct = list(0. for i in range(10))
class_total = list(0. for i in range(10))
with torch.no_grad():
    for data in testloader:
        images, labels = data
        outputs = net(images)
        _, predicted = torch.max(outputs, 1)
        c = (predicted == labels).squeeze()
        for i in range(4):
            label = labels[i]
            class_correct[label] += c[i].item()
            class_total[label] += 1


for i in range(10):
    print('Accuracy of %5s : %2d %%' % (
        classes[i], 100 * class_correct[i] / class_total[i]))
```

# Training on GPU

Just like how you transfer a Tensor onto the GPU, you transfer the neural net onto the GPU

```
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")

# Assuming that we are on a CUDA machine, this should print a CUDA device:

print(device)


out:
cuda:0
```
说明是在gpu设备上的


下面这些方法会遍历所有的模块，把参数和缓存都变成CUDA Tensor
```
net.to(device)
```

记住你必须每一步都把input和label都变成GPU支持的
```
inputs, labels = data[0].to(device), data[1].to(device)
```


后面还有一些好的例子，可以看官网：
https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html