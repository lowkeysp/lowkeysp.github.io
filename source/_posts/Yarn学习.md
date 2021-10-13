---
title: Yarn学习
date: 2021-10-13 11:07:49
tags:
---

<meta name="referrer" content="no-referrer" />

# Yarn的基本架构
由Resource Manager，Node Manager，Application Master，Container组成

![捕获.PNG](http://ww1.sinaimg.cn/large/006eDJDNly1gvdilz0e48j61h30r5at402.jpg)

# Yarn调度器

## FIFO调度器
FIFO调度器将应用放置在一个队列中，然后按照提交的顺序（先进先出）运行应用。首先为队列中第一个应用的请求分配资源，第一个应用的请求被满足后再依次为队列中下一个应用服务。

FIFO调度器的优点是：简单易懂，不需要任何配置。但是不适合共享集群。大的应用会占用集群中的所有资源，所以每个应用必须等待直到轮到自己运行。在一个共享集群中，更适合容量调度器和公平调度器。这两种调度器都允许长时间运行的作业能够及时完成，同时也允许正在进行较小临时查询的用户能够在合理时间内得到返回结果。

## 容量调度器
Hadoop默认是容量调度器

容量调度器允许多个组织共享一个hadoop集群，每个组织可以分配到全部集群资源的一部分。每个组织被配置一个专门的队列，每个队列被配置为可以使用一定的集群资源。队列还可以进一步按照层次划分，这样每个组织内的不同用户能供共享该组织队列所分配的资源。在一个队列里，使用FIFO调度策略进行调度。

单个作业所使用的资源不会超过其队列容量。然而，如果一个队列中有多个作业，并且队列资源不够的话，如果仍有可用的空闲资源，那么容量调度器可能会将空余的资源分配给队列中的作业，哪怕这会超出队列容量，这称为“弹性队列”

正常操作时，容量调度器不会通过强行中止来抢占容器，因此，如果一个队列一开始资源够用，然后随着需求增长，资源开始不够用时，那么这个队列就只能等着其他队列释放容器资源。缓解这种情况的方法是：对队列设置一个最大容量限制，这样这个队列就不会过多侵占其他队列的容量了。

## 公平调度器

公平调度器旨在为所有运行的应用公平分配资源。包括同一个队列中的应用实现资源公平共享，多个队列间实现公平共享。

如何在队列之间公平共享？想象两个用户A和B，分别拥有自己的队列。A启动一个作业，在B没有需求时，A会分配到全部可用资源；当A的作业仍在运行时B启动一个作业，一段时间后，每个作业都用到了一半的集群资源。这时，如果B启动第二个作业且其他作业仍在运行，那么第二个作业将和B的其他作业（这里是第一个）共享资源，因此B的每个作业将占用四分之一的集群资源，而A仍继续占用一半的集群资源。


# Yarn常用命令
* 展示运行中任务的详细信息
`yarn application-list`

* 筛选出某个状态的任务
`yarn application-list-appStates` 

* kill某一个任务
`yarn application -kill <applicationId>`

* 查看applicationd的logs
`yarn logs-applicationId <applicationId>`

* 查看Container日志
`yarn logs-applicationId <applicationId> -containerId <containerId>`

* 查看尝试运行的任务
`yarn applicationattempt -list <applicationId>`

* 查看application所用的容器
`yarn container -list <applicationattemptId>`

* 打印Container状态
`yarn container -status <ContainerId>`

* 列出所有节点
`yarn node -list all`

* yarn rmadmin 刷新配置,在运行时如果对队列配置文件有更改时，可用这个命令刷新配置
`yarn rmadmin -refreshQueues`

* yarn queue状态
`yarn queue -status default`
