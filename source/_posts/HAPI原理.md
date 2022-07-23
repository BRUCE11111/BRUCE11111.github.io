---
title: HAPI原理
top: false
cover: false
toc: true
mathjax: true
date: 2021-07-19 20:17:52
password:
summary:
tags: HAPI
categories: HAPI
---

HAPI 主要是两种东西构成的：

第一个：数据模型。

医疗相关数据的数据模型。这些数据模型需要放到java中定义的数据结构里面。数据结构的实现是基于HL7机构约定的标准。为了使用方便，HAPI 将JAVA中的一些基本数据类型比如string boolean 等重新封装成 stringType BooleanType 等，然后将数据塞到这些数据类型中。 里面的其他复合类型，比如地址Address这个数据结构，使用这些基本数据类型进行了组合封装。这样符合了HL7 结构定义的标准。

第二个：restful API 对于交换这种医疗数据。

这个其实就是servlet的实现了，类似于springboot中的controller那种，servlet对请求进行转发到对应的方法。

主要模块：

用java实现了HL7 定义的数据标准模型。

实现了数据转换 （JSON + XML）

服务器 server framework

客户端 client framework



