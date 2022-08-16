---
title: System类源码分析
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-31 11:38:47
password:
summary:
tags: Java基础
categories: Java
---

## The Security Manager

1. 定义

官方：

A security manager is an object that defines a security policy for an application. This policy specifies actions that are unsafe or sensitive. Any actions not allowed by the security policy cause a `SecurityException` to be thrown. An application can also query its security manager to discover which actions are allowed.

理解：

它是一个应用的安全策略。这个策略指定当前的动作是安全or不安全or敏感。

2. 示例

官方：

Typically, a web applet runs with a security manager provided by the browser or Java Web Start plugin. Other kinds of applications normally run without a security manager, unless the application itself defines one. If no security manager is present, the application has no security policy and acts without restrictions.

理解：

web 程序中的安全管理类是由浏览器或者启动插件提供的。其他应用程序通常没有安全管理类，除非我们自己去定义。

在







