---
title: mybatis-plus
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-07 15:59:52
password:
summary:
tags: Java基础
categories: 
  - Java
---

## 1. 介绍

官方地址：[MyBatis-Plus (baomidou.com)](https://baomidou.com/)



## 2. 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1</version>
</dependency>
```



## 3.注解

### 3.1 @TableId

设置主键映射。value设置映射。

type 设置主键的生成策略。

| 值          | 描述                            |
| ----------- | ------------------------------- |
| AUTO        | 数据库自增                      |
| NONE        | MP set主键，雪花算法实现        |
| INPUT       | 需要开发者手动赋值              |
| ASSIGN_ID   | MP分配ID，Long、Integer、String |
| ASSIGN_UUID | 分配UUID，String                |

INPUT 如果开发者手动没有赋值，则数据库通过自增的方式给主键赋值，如果开发者手动赋值，则存入该值。

AUTO默认就是数据库自增，开发者无需赋值。

ASSIGN_ID MP 自动赋值，雪花算法。

ASSIGN_UUID 主键的数据类型必须是String，自动生成UUID进行赋值。





### 3.2 @TableField

设置类中属性与表中名字映射。

* exist

是否为数据库字段。false，告诉mybatisplus查的时候不要把这个字段带上。

* fill

表示是否自动填充，将对象存入数据库的时候，由MyBatis Plus自动给某些字段赋值，create_time、update_time。

```java
@TableField(fill = FieldFill.INSERT)
private Date createTime;
```

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.setFieldValByName("crateTime", new Date(), metaObject);
    }
    
    @Override
    public void updateFill(MetaObject metaObject) {
        
    }
}
```



### 3.3 @TableName

设置类名字与表中名字映射





### 3.4 @Version

标记**乐观锁**，通过version字段来保证数据的安全性，当修改数据的时候会以version作为条件，当条件成立的时候，才会修改成功。

version=1。 这样当多个线程修改的时候保证修改的成功。(主要是修改操作)

线程1：update ... set version = 2 where version = 1

线程2：update ... set version = 2 where version = 1

1、数据库表添加version字段，默认为1。

2、实体类添加version成员变量，并且添加@Version注解

3、添加配置类，Version字段会自动找这个配置类

```java
@Configuration
public class MyBatisPlusConfig {
    @Bean
    public OptimisticLockerIntegerceptor optimisticLockerIntegerceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```



### 3.5 @EnumValue

1. 添加配置文件中信息

```yaml
mybatis-plus:
	type-enums-package:
		com.southwind.mybatisplus.enums//自己的类名
```

2. 添加注解类

```java
public enum StatusEnum {
    WORK(1, "上班");
    REST(0,"休息");
    
    StatusEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }
    
    @EnumValue
    private Integer code;
    private String msg;
    
}
```

可以将数据库中的1映射成上班，0休息。



第二种实现方式：

实现接口IEnun

```java
public enum AgeEnum implements IEnum<Integer> {
    ONE(1, "一岁");
    TWO(2, "两岁");
    THREE(3, "三岁");
    private Integer code;
    private Integer msg;
    AgeEnum (Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }
    
    @Override
    public Integer getValue() {
        return this.code;
    }
    
}
```

### 3.6 TableLogic

1. 实体类中添加映射逻辑删除。

```java
@TableLogic
private Integer deleted;
```

2、配置文件中

```yaml
mybatis-plus:
global-config:
	db-donfig:
		logic-not-delete-value: 0
		logic-delete-value : 1
```

## 4. CRUD

### 4.1 查询







## 5. 自动生成

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.0</version>
</dependency>

<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity</artifactId>
    <version>1.7</version>
</dependency>
```

1. 自动生成器的包和velocity都需要，因为自动生成器根据velocity模板导入的。

2. 启动类





### 5.1 工程依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!--mybatis-plus-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>

<!--mysql-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<!-- velocity 模板引擎, Mybatis Plus 代码生成器需要 -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>

<!--swagger-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
</dependency>
<!--swagger ui-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
</dependency>
```





## 6. 自定义SQL、多表关联













### 7. SpringBoot + Mybatis Plus 打包直接发布









