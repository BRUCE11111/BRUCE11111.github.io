---
title: raft算法
top: false
cover: false
toc: true
mathjax: true
date: 2021-09-22 10:46:31
password:
summary:
tags: 分布式
categories: 碎片知识
---

# Raft算法

### 1、为什么

需要跨网络的机器之间协调一致的工作。

应对网络的不可靠以及节点的失效。

### 2、 是什么

组织机器使其状态最终一致并允许局部失败的算法。

Paxos算法由来已久，目前是功能和性能最完善的一致性算法，然而他难以理解和实现。raft简化了paxos，以便于理解为目标。尽量提供和paxos一样的功能和性能。

![image-20210922161805776](D:\1\blog\source\_posts\raft算法\image-20210922161805776.png)

​																						raft 状态机



