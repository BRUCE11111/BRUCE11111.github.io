# Mybatis

## 1. 配置maven

### 1.1 下载地址

[maven 官网下载地址](http://maven.apache.org/download.html )

参考环境配置 

https://www.cnblogs.com/telwanggs/p/10820701.html



mybatis 中文文档 https://mybatis.org/mybatis-3/zh/configuration.html

### 1.2 编写mybatis配置文件

XML配置文件`configuration XML`中包含了对MyBatis系统的核心设置。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

### 1.3 编写代码

* 实体类
* Dao接口
* 接口实现类

接口实现由原来的`UserDaoImpl`转换为`Mapper`配置文件。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.dao.UserDao">
    <select id="getUserList" resultType="com.kuang.pojo.User ">
        select * from mybatis.user
    </select>
</mapper>
```

### 1.4 测试

* junit测试

注意点：报错

org.apache.ibatis.binding.BindingException: Type interface com.kuang.dao.UserDao is not known to the MapperRegistry.





### 1.5 知识点

1. 从SqlSessionFactory中获取SqlSession。

2. 命名空间要写，绑定接口。写完全限定名。

3. <font color = "red">SqlSessionFactoryBuilder</font>

   这个类可以被实例化、使用和丢弃，一旦创建SqlSessionFactory就不再需要他了。因此它实例的最佳作用是方法作用域。

4. SqlSessionFactory

   * 可以想象为数据库连接池。

   SqlSessionFactory一旦创建就应该在运行期间一直存在。默认单例模式或静态单例模式。（适合全局）。

5. SqlSession

   SqlSesion不是线程安全的，不能被共享。每次收到的HTTP请求，就可以打开一个SqlSession，返回一个相应就应该关闭它。

   ```java
   try (SqlSession session = sqlSessionFactory.openSession()){
       你的应用逻辑代码
   }
   ```

![](D:\1\奇技淫巧\photoDon't delete\Spring\mybatisFactory.PNG)



## 2.CRUD

### 1、 namespace

namespace中的包名要和Dao/mapper接口中的一致。

### 2、select

选择，查询语句

* id:对应的namespace中的方法。
* returnType:Sql语句执行的返回值！
* parameterType：参数类型



## 3. Map

不需要知道数据库中有什么，只需要查对应的字段。

数据库中表或者字段参数过多，应当考虑使用Map。



## 4.配置解析

#### 4.1 核心配置文件

  configuration（配置）

```xml
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

#### 4.2 环境配置

Mybatis 事务管理器有两种：JDBC和MANAGED。默认JDBC。



datasource 连接数据库。内置三种数据源类型 type = UNPOOLED | POOLED| JNDI。默认连接池POOLED。

学会使用配置多套运行环境。

池子的解释：用完可以回收。

#### 4.3 属性

可以通过properties来提供多种实现方式。

属性都是可外部动态替换的，可在典型的Java属性文件中配置，亦可通过properties元素的子元素来改造。



#### 4.4 别名

* 类型别名是为Java类型设置一个短的名字。
* 存在的意义用来减少类完全限定名的冗余。

```xml
<typeAliases>
        <typeAlias type="com.kuang.pojo.User" alias="User"/>
</typeAliases>
```

扫描实体类的包，它的默认别名就为这个类的 类名，首字母小写！

```xml
<typeAliases>
        <package type="com.kuang.pojo"/>
</typeAliases>
```



在实体类比较少的时候使用第一种方式，如果实体类比较多，使用第二种。第一种可以DIY， 第二种则不行。



```java
@Alias("user")
public class User{
    
}
```



#### 4.5 Mappers 映射

MapperRegistry：注册绑定mappers文件。

第一种方式。mapper文件可以随便放。

```xml
<mappers>
    <mapper resource="com/kuang/dao/UserMapper.xml"/>
</mappers>
```

第二种方式。类的方式。

```xml
<mappers>
     <mapper class="com.kuang.dao.UserMapper"/>
 </mappers>
```

* 接口和Mapper配置文件必须在同一个包下。
* 接口和Mapper配置文件必须同名。



第三种方式。package方式

```xml
<mappers>
        <package name="com.kuang.dao"/>
</mappers>
```

注意方式：

* 接口和Mapper配置文件必须在同一个包下。
* 接口和Mapper配置文件必须同名。

## 5. 解决属性名和字段名不一致的问题



### 5.1 结果集映射

```xml
id name pwd
id name password
```

```xml
<resultMap id="UserMap" type="User">
<!--        column 数据库中的字段 property 实体类属性-->
        <result column="pwd" property="password"/>
    </resultMap>
    <select id="getUserById" resultMap="UserMap" resultType="com.kuang.pojo.User">
        select * from mybatis.user1 where id=#{id};
    </select>
```



## 6.日志

如果一个数据库操作出现了异常，需要排错。日志就是最好的助手。

曾经：sout、debug

现在：日志工厂！

logImpl 指定Mybatis所用日志的具体实现。

| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |
| ------- | ----------------------------------------------------- | ------------------------------------------------------------ | ------ |
|         |                                                       |                                                              |        |



具体使用哪一个日志实现，默认未设置日志工厂。

标准是STDOUT_LOGGING

### 6.1 Log4j

先导入Log4j的包。

```xml
<dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
```

* 通过一个配置文件来配置

```properties
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关配置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold = DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern = [%c]-%m%n


# 将文件输出的相关配置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File = ./log/kuang.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

# 日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

* 配置log4j为日志的实现

  ```xml
  <settings>
          <setting name="logImpl" value="Log4j"/>
      </settings>
  ```

  

* 使用

1. 在要使用Log4j的类中导入包。

```java

import org.apache.log4j.Logger;
```

2. 日志对象加载参数为当前类的class。

```java
 static Logger logger = Logger.getLogger(UserDaoTest.class);
```

3. 日志级别

```java
logger.info("");
logger.debug("");
logger.error("");
```

## 7.分页

常见的就是使用mysql里面的limit关键字.比如：

```mysql
SELECT * FROM limit 0,2;
```

使用mybatis实现分页：

1. 接口

```java
//分页
    List<User> getUserByLimit(Map<String, Integer> map);
```



2. Mapper.XML

```xml
<select id="getUserByLimit" parameterType="map" resultType="user">
        select * from mybatis.user1 limit #{startIndex},${pageSize};
    </select>
```

3. 测试



### 7.2 RowBounds

不再使用SQL实现分页。

1. 接口

```java
List<User> getUserByRowBounds();
```



2. mapper.xml

```
<select id="getUserByRowBounds" resultMap="UserMap">
    select * from mybatis.user1;
</select>
```

1. 测试

### 7.3 分页插件

PageHelper

## 8.使用注解开发

### 8.1 面向接口编程

* 定义（约束与规范）与实现的分离

* 有一些mapper不用xml配置，直接使用注解。



1. 注解在接口上实现

```java
@Select("select * from user1")
    List<User> getUsers();
```

2. 在核心配置文件中绑定接口

```xml
<mappers>
<!--        <mapper resource="com/kuang/dao/UserMapper.xml"/>-->
        <mapper class="com.kuang.dao.UserMapper"/>
<!--        <package name="com.kuang.dao"/>-->
    </mappers>
```



3. 测试

```java
@Test
    public void test(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = mapper.getUsers();

        for (User user : users) {
            System.out.println(user);
        }

        sqlSession.close();
    }
```



### 8.2 CURD

* 可以在工具类创建的时候实现自动提交事务。
* @Param（"name"）。SQL引用的就是这个起的名字

## 9. Mybatis执行流程





## 10. Lombok

通过注解消除冗余代码

缺点：1、不支持多种参数构造器的重载 2、降低了源代码的可读性和完整性

1. 安装插件
2. 导入lombok包(pom.xml中配置)
3. 实体类加注解

```java
@Data:无参构造、get、set、tostring、hashcode、equals
@ALLArgsConstructor
@NoArgsConstructor
@EqualsAndHashCode
@ToString
@Getter
```

## 11.多对一

* 多个学生对应一个老师



### 11.1按照查询嵌套处理

```xml
<mapper namespace="com.kuang.dao.StudentMapper">
    <select id="getStudent" resultMap="StudentTeacher">
        select * from student
    </select>

    <resultMap id="StudentTeacher" type="com.kuang.pojo.Student">
<!--        对象使用association-->
        <association property="teacher" column="tid" javaType="com.kuang.pojo.Teacher" select="getTeacher"/>
    </resultMap>
    <select id="getTeacher" resultType="com.kuang.pojo.Teacher">
        select * from teacher where id = #{id}
    </select>

</mapper>
```



### 11.2 按照结果嵌套查询

```xml
<select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid, s.name sname, t.name tname from student s, teacher t where s.tid = t.id;
    </select>

    <resultMap id="StudentTeacher2" type="com.kuang.pojo.Student">
        <!--        对象使用association-->
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="com.kuang.pojo.Teacher">
            <result property="name" column="tname"/>
        </association>
    </resultMap>

```

### 11.3 一对多处理

同上，其实是类似的



### 小结

javaType && ofType

javaType 用来指定实体类中的属性的类型

ofType用来指定映射到List或集合中的pojo类型，泛型中的约束类型。



## 12. 动态SQL

动态SQL根据不同的条件生成不同的sql语句。

- if

### if

使用动态 SQL 最常见情景是根据条件包含 where 子句的一部分。比如：

```xml
<select id="findActiveBlogWithTitleLike"
        resultType="Blog">
    SELECT * FROM BLOG
    WHERE state = ‘ACTIVE’
    <if test="title != null">
        AND title like #{title}
    </if>
</select>
```

- choose (when, otherwise)

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

- trim (where, set)

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE
  <if test="state != null">
    state = #{state}
  </if>
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

注意，set语句别忘了后面加,

### SQL片段

1. 使用SQL标签抽取公共的部分

```xml
<sql id="if-title-author">
    <if test="title != null">
        title = #{title}
    </if>
    <if test="author != null">
        author = #{author}
    </if>
</sql>
```

2. 在需要的地方使用incllude标签引用即可

```xml
<select id="queryBlogIF" parameterType="map" resultType="com.kuang.pojo.Blog">
    select * from mybatis.blog
    <where>
        <include refid="if-title-author"></include>
    </where>
</select>
```





- foreach

是对一个集合进行遍历，通常构建IN条件语句的时候。

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```



## 13. 缓存



```
查询 ： 连接数据库，耗费资源！
	一次查询的结果，给他暂时存在一个直接取到的地方！--> 内存： 缓存
	
我们再次查询相同数据的时候，直接走缓存，就不用走数据库了。
```



1. 什么是缓存？
   * 放在内存中的临时数据
   * 将用户经常查的数据放在内存中可以提高性能。



* Mybatis默认两级缓存，默认一级缓存。

* 二级缓存需要手动开启配置。他是基于namespace级别的缓存。

* 为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过Cache接口自定义二级缓存。



### 13.1 一级缓存

一级缓存也叫本地会话缓存。

* 与数据库同一次会话期间查询到的数据会放在本地缓存中。
* 以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库。

缓存失效的情况：

1. 所有增删改都要缓冲失效。（增删改有可能会改变数据，所以缓存需要失效）
2. 查询不同的mapper.xml
3. 查询不同的数据。
4. 手动清理缓存。



小结：一级缓存默认是开启的，只在一次SqlSession中有效，也就是拿到连接到关闭连接。



### 13.2 二级缓存

* 工作机制
  * 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中。
  * 如果当前会话关闭，这个会话对应的一级缓存就没了。但是我们希望会话关闭了，一级缓存中的数据被保存到二级缓存中。
  * 新的会话查询信息，就可以从二级缓存中获取内容。
  * 不同的mapper查出的数据会放在自己的缓存map中。



2. 缓存清除策略

* LRU 最近最少使用
* FIFO 先进先出
* SOFT 软引用
* WEAK 弱引用，更积极地基于垃圾收集器状态和弱引用规则移除对象。

步骤：

1. 开启全局缓存。`cacheEnabled = true`

```xml
 <setting name="cacheEnabled" value="true"/>
```

2. 在Mapper.xml中加入

   ```xml
   <cache/>
   ```

### 13.3 缓存原理

再次查询时会先找二级缓存，再一级，如果两个都没有对应的数据，再从数据库中找。

### 13.4 自定义缓存ehcache

# 面试部分

## 1. 一部分

### 1.1 和jdbc的区别

jdbc是java提供的操作数据库的API。

mybatis消除了几乎所有的JDBC代码和参数的手工设置以及对应的结果集的检索封装。

mybatis是对JDBC的封装，相对于JDBC，Mybatis有以下优点：

1. 优化获取和释放
2. SQL统一管理，对数据库进行存取操作。
3. 生成动态SQL语句
4. 能够对结果集进行映射

### 1.2 为什么是半自动ORM映射工具？

* Hibernate 属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。

* mybatis在查询关联对象时需要手动编写sql来完成，所以是半自动ORM映射工具。

### 1.3 一个xml映射文件，都会有一个Dao接口与之对应。dao接口工作原理以及dao的方法参数不同是否能够重载？

* Dao接口，就是常说的Mapper接口，接口的全限名，就是映射文件中的namespace值，接口的方法名，就是映射文件中的MappedStatement的id值，接口方法内的参数就是传递给sql的参数。
* Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼装字符串作为key值，可以唯一定位一个MappedStatement。

Dao接口的工作原理是JDK动态代理，mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。

原理可以参考：[[MyBatis\]DAO层只写接口，不用写实现类 - 左正 - 博客园 (cnblogs.com)](https://www.cnblogs.com/soundcode/p/6497291.html)

### 1.4 和ibatis比较大的几个改进？

1. 有接口绑定，包括注解绑定sql和xml绑定sql。
2. 动态sql由原来的节点配置编程了OGNL表达式。
3. 在一对一，一对多的时候引进了association，在一对多的时候引入了collection节点，不过都是在resultMap里面配置

### 1.5 接口绑定有几种实现方式，分别如何实现？

1. 通过xml。指定xml映射文件里面的namespace必须为接口的全路径名。
2. 通过注解。通过在接口的方法上添加@select等注解。

### 1.6 mybatis如何进行分页？分页插件原理？

使用`RowBounds`对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。

### 1.7 Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？

- Mybatis动态sql可以让我们在Xml映射文件内，**以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能**。
- Mybatis提供了9种动态sql标签：trim|where|set|foreach|if|choose|when|otherwise|bind。
- 其执行原理为，使用OGNL从sql参数对象中计算表达式的值，**根据表达式的值动态拼接sql，以此来完成动态sql的功能**。

### 1.8 Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？

**如果配置了namespace那么当然是可以重复的，因为我们的Statement实际上就是namespace+id**。

如果没有配置namespace的话，那么相同的id就会导致覆盖了。

### 2. 二部分

参考至：[MyBatis面试题（2020最新版）_ThinkWon的博客-CSDN博客_mybatis面试题](https://blog.csdn.net/ThinkWon/article/details/101292950)

### 2.1 MyBatis编程步骤是什么样的？
1、 创建SqlSessionFactory

2、 通过SqlSessionFactory创建SqlSession

3、 通过sqlsession执行数据库操作

4、 调用session.commit()提交事务

5、 调用session.close()关闭会话

### 2.2 mybatis工作原理

![MyBatis工作原理](Mybatis/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL0pvdXJXb24vaW1hZ2UvbWFzdGVyL015QmF0aXMlRTYlQTElODYlRTYlOUUlQjYlRTYlODAlQkIlRTclQkIlOTMvTXlCYXRpcyVFNSVCNyVBNSVFNCVCRCU5QyVFNSU4RSU5RiVFNyU5MCU4Ni5wbmc)

1）读取 MyBatis 配置文件：mybatis-config.xml 为 MyBatis 的全局配置文件，配置了 MyBatis 的运行环境等信息，例如数据库连接信息。

2）加载映射文件。映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件 mybatis-config.xml 中加载。mybatis-config.xml 文件可以加载多个映射文件，每个文件对应数据库中的一张表。

3）构造会话工厂：通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。

4）创建会话对象：由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。

5）Executor 执行器：MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。

6）MappedStatement 对象：在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息。

7）输入参数映射：输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型。输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程。

8）输出结果映射：输出结果类型可以是 Map、 List 等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程。

### 2.3 Mybatis都有哪些Executor执行器？它们之间的区别是什么？
Mybatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

`SimpleExecutor`：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。

`ReuseExecutor`：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。

`BatchExecutor`：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。

### 1.4 Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？
Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。

它的原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。

### 2.5 #{}和${}的区别
#{}是占位符，预编译处理；\${}是拼接符，字符串替换，没有预编译处理。

1. Mybatis在处理#{}时，#{}传入参数是以字符串传入，会将SQL中的#{}替换为?号，调用PreparedStatement的set方法来赋值。

2. Mybatis在处理时 ， 是 原 值 传 入 ， 就 是 把 {}时，是原值传入，就是把时，是原值传入，就是把{}替换成变量的值，相当于JDBC中的Statement编译

3. 变量替换后，#{} 对应的变量自动加上单引号 ‘’；变量替换后，${} 对应的变量不会加上单引号 ‘’

4. #{} 可以有效的防止SQL注入，提高系统安全性；${} 不能防止SQL 注入

5. #{} 的变量替换是在DBMS 中；${} 的变量替换是在 DBMS 外

### 2.6 Mapper 编写有哪几种方式？
第一种：接口实现类继承 SqlSessionDaoSupport：使用此种方法需要编写mapper 接口，mapper 接口实现类、mapper.xml 文件。

（1）在 sqlMapConfig.xml 中配置 mapper.xml 的位置

```xml
<mappers>
    <mapper resource="mapper.xml 文件的地址" />
    <mapper resource="mapper.xml 文件的地址" />
</mappers>
```

（2）定义 mapper 接口

（3）实现类集成 SqlSessionDaoSupport

mapper 方法中可以 this.getSqlSession()进行数据增删改查。

（4）spring 配置

```xml
<bean id=" " class="mapper 接口的实现">
    <property name="sqlSessionFactory"
    ref="sqlSessionFactory"></property>
</bean>
```


第二种：使用 org.mybatis.spring.mapper.MapperFactoryBean：

（1）在 sqlMapConfig.xml 中配置 mapper.xml 的位置，如果 mapper.xml 和mappre 接口的名称相同且在同一个目录，这里可以不用配置。

```xml
<mappers>
    <mapper resource="mapper.xml 文件的地址" />
    <mapper resource="mapper.xml 文件的地址" />
</mappers>
```

（2）定义 mapper 接口：

（3）mapper.xml 中的 namespace 为 mapper 接口的地址

（4）mapper 接口中的方法名和 mapper.xml 中的定义的 statement 的 id 保持一致

（5）Spring 中定义

```xml
<bean id="" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="mapperInterface" value="mapper 接口地址" />
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

第三种：使用 mapper 扫描器：

（1）mapper.xml 文件编写：

mapper.xml 中的 namespace 为 mapper 接口的地址；

mapper 接口中的方法名和 mapper.xml 中的定义的 statement 的 id 保持一致；

如果将 mapper.xml 和 mapper 接口的名称保持一致则不用在 sqlMapConfig.xml中进行配置。

（2）定义 mapper 接口：

注意 mapper.xml 的文件名和 mapper 的接口名称保持一致，且放在同一个目录

（3）配置 mapper 扫描器：

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="mapper 接口包地址
    "></property>
    <property name="sqlSessionFactoryBeanName"
    value="sqlSessionFactory"/>
</bean>
```


### 2.7 使用扫描器后从 spring 容器中获取 mapper 的实现对象。

简述Mybatis的Xml映射文件和Mybatis内部数据结构之间的映射关系？
答：Mybatis将所有Xml配置信息都封装到All-In-One重量级对象Configuration内部。在Xml映射文件中，<parameterMap>标签会被解析为ParameterMap对象，其每个子元素会被解析为ParameterMapping对象。<resultMap>标签会被解析为ResultMap对象，其每个子元素会被解析为ResultMapping对象。每一个<select>、<insert>、<update>、<delete>标签均会被解析为MappedStatement对象，标签内的sql会被解析为BoundSql对象。

### 2.8 缓存
Mybatis的一级、二级缓存
1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存。

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置<cache/> ；

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

[彻底搞明白mybatis缓存（上） - SegmentFault 思否](https://segmentfault.com/a/1190000038544664)



### 2.9 分页原理 pagehelper

PageHelper首先将前端传递的参数保存到page这个对象中，接着将page的副本存放入ThreadLoacl中，这样可以保证分页的时候，参数互不影响，接着利用了mybatis提供的拦截器，取得ThreadLocal的值，重新拼装分页SQL，完成分页。

[浅析pagehelper分页原理_王大锤的博客-CSDN博客_pagehelper分页原理](https://blog.csdn.net/qq_21996541/article/details/79796117)

