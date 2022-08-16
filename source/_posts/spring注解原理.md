---
title: spring注解原理
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-31 15:36:33
password:
summary:
tags: Spring
categories: Java
---

# Spring 注解驱动开发

## 1. 配置文件



1、配置文件的形式中，标注了`@component-scan`,`@Controller`,`@Service`,`@Repository`,`@Component`就会自动扫描包。

2、`@ComponentScan(value="com.atguigu")`,



### 1.1 配置







## 2. 常用注解

`@Controller` 注解的bean会被spring-mvc框架所使用。
`@Repository` 会被作为持久层操作（数据库）的bean来使用
如果想使用自定义的组件注解，那么只要在你定义的新注解中加上@Component即可：

```java
@Component 
@Scope("prototype")
public @interface ScheduleJob {...}123
```

这样，所有被`@ScheduleJob`注解的类就都可以注入到spring容器来进行管理。我们所需要做的，就是写一些新的代码来处理这个自定义注解（译者注：可以用反射的方法），进而执行我们想要执行的工作。

`@Component`就是跟`<bean>`一样，可以托管到Spring容器进行管理。

`@Service`, `@Controller` , `@Repository` = {`@Component` + 一些特定的功能}。这个就意味着这些注解在部分功能上是一样的。

当然，下面三个注解被用于为我们的应用进行分层：

`@Controller`注解类进行前端请求的处理，转发，重定向。包括调用Service层的方法 （requestMapping + componnet)
`@Service`注解类处理业务逻辑
`@Repository`注解类作为DAO对象（数据访问对象，Data Access Objects），这些类可以直接对数据库进行操作
有这些分层操作的话，代码之间就实现了松耦合，代码之间的调用也清晰明朗，便于项目的管理；假想一下，如果只用`@Controller`注解，那么所有的请求转发，业务处理，数据库操作代码都糅合在一个地方，那这样的代码该有多难拓展和维护。

### 总结

`@Component`, `@Service`, `@Controller`, `@Repository`是spring注解，注解后可以被spring框架所扫描并注入到spring容器来进行管理
`@Component`是通用注解，其他三个注解是这个注解的拓展，并且具有了特定的功能
`@Repository`注解在持久层中，具有将数据库操作抛出的原生异常翻译转化为spring的持久层异常的功能。
`@Controller`层是spring-mvc的注解，具有将请求进行转发，重定向的功能。
`@Service`层是业务逻辑层注解，这个注解只是标注该类处于业务逻辑层。
用这些注解对应用进行分层之后，就能将请求处理，义务逻辑处理，数据库操作处理分离出来，为代码解耦，也方便了以后项目的维护和开发。

### 1.@RequestMapping

1）遍历Handler中的所有方法，找出其中被@RequestMapping注解标记的方法。

2）遍历这些方法，生成RequestMappingInfo实例。

3）将RequestMappingInfo实例以及处理器方法注册到缓存中。

当请求到达时，去urlMap中找匹配的url，以及获取对应的mapping实例，然后去handlerMethods中获取匹配HandlerMethod实例。

### 2. @Autowired

autowired是用来装配bean的，和bean的初始化有关。当执行refresh方法后，在触发了bean的初始化后，创建BeanWrapper，它的内部成员变量wrappedObject存放的就是实例化的MyService对象，然后进行属性注入。

获取需要处理的目标类，传入目标类参数，通过反射获取该类的所有字段，遍历每行一个字段，判断是否被autowired修饰，返回所有带有autowired注解修饰的一个集合。有了目标类和刚才返回的这个需要注入的集合。

注入：

* 先使用反射技术对对象进行实例化和赋值，然后这个bean就生成了。

* 根据bean的名字去拿到bean进行注入。



# Spring

Spring是一个轻量级的开放源代码的J2EE应用程序框架。

















