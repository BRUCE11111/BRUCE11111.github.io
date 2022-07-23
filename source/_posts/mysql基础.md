---
title: mysql基础
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-10 10:58:01
password:
summary:
tags: mysql
categories: mysql
---

## 1.安装

# 1 centos

### 1.1 centos mysql 5.7

> CentOS 7的默认yum仓库中并没有MySQL5.7，我们需要手动添加，好在MySQL官方提供了仓库的地址，所以我们能够比较简单地安装MySQL。
>
> 本文我们将介绍CentOS 7下MySQL5.7的安装。

1. 添加Mysql5.7仓库

```bash
sudo rpm -ivh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

1. 确认Mysql仓库成功添加

```bash
sudo yum repolist all | grep mysql | grep enabled
```

如果展示像下面,则表示成功添加仓库:

```bash
mysql-connectors-community/x86_64  MySQL Connectors Community    enabled:     51
mysql-tools-community/x86_64       MySQL Tools Community         enabled:     63
mysql57-community/x86_64           MySQL 5.7 Community Server    enabled:    267
```

1. 开始安装Mysql5.7

```bash
sudo yum -y install mysql-community-server
```

centos8.2安装mysql8.0时报错Error:Unable to find a match: mysql-community-server

```
yum module disable mysql
yum -y install mysql-community-server
```



1. 

2. 启动Mysql

   1. 启动

   ```bash
   sudo systemctl start mysqld
   ```

   1. 设置系统启动时自动启动

   ```bash
   sudo systemctl enable mysqld
   ```

   1. 查看启动状态

   ```bash
   sudo systemctl status mysqld
   ```

3. Mysql的安全设置

   CentOS上的root默认密码可以在文件/var/log/mysqld.log找到，通过下面命令可以打印出来

   ```bash
   cat /var/log/mysqld.log | grep -i 'temporary password'
   ```

   执行下面命令进行安全设置，这个命令会进行设置root密码设置，移除匿名用户，禁止root用户远程连接等

   ```bash
   mysql_secure_installation
   ```

4. 设置数据库编码为utf8

   1. 打开配置文件

   ```bash
   sudo vim /etc/my.cnf
   ```

   1. 在[mysqld]，[client]，[mysql]节点下添加编码设置

   ```bash
   [client]
   default-character-set=utf8
   
   [mysql]
   default-character-set=utf8
   
   [mysqld]
   collation-server = utf8_unicode_ci
   init-connect='SET NAMES utf8'
   character-set-server = utf8
   ```

   1. 重启Mysql即可

   ```bash
   sudo systemctl restart mysqld
   ```

#### 1.1.2 更改用户名和密码

1、修改 `/etc/my.cnf`，在 [mysqld] 小节下添加一行：`skip-grant-tables=1`

这一行配置让 mysqld 启动时不对密码进行验证

2、重启 mysqld 服务：`systemctl restart mysqld`

3、使用 root 用户登录到 `mysql：mysql -u root`

注释：当看到密码输入时直接按return

添加新密码

1、切换到mysql数据库，更新 user 表：

`use mysql`

`update user set authentication_string = password('新密码'), password_expired = 'N', password_last_changed = now() where user = 'root';`

在之前的版本中，密码字段的字段名是 password，5.7版本改为了 `authentication_string`

恢复使用root密码

1、退出 mysql，编辑 `/etc/my.cnf `文件，删除 skip-grant-tables=1 的内容

2、重启 mysqld 服务，再用新密码登录即可

### 2.1安装MySQL8.0

参考：

[(21条消息) centos8安装mysql8.0.22教程(超详细)_上善若水滴世界的博客-CSDN博客_centos8安装mysql8.0](https://blog.csdn.net/qq_39150374/article/details/112471108)



在yum install 版本后面加上 --nogpgcheck，即可绕过GPG验证成功安装。比如

yum install mysql-community-server --nogpgcheck 作者：又双叒叕变帅了 https://www.bilibili.com/read/cv15259704 出处：bilibili

#### 2.1.1 修改密码

```mysql
alter user 'root'@'localhost' identified by '';
```





## 2. 备份和恢复

**备份方法**

```
mysqldump -uroot -p000 db table > db.sql
```


**还原方法**

```mysql
mysql -uroot -p000 < db.sql #直接恢复整个数据库
mysql -uroot -p000 db_tencent_health_data < t_doctor_20210607.sql #在这个数据库中建立这张表
```


 参考： 

1.拷备文件      :  (保证数据库没有写操作(可以给表上锁定))直接拷贝文件不能移植到其它机器上，除非你正在拷贝的表使用MyISAM存储格式 
2.mysqldump   :  mysqldump生成能够移植到其它机器的文本文件 

例: 
备份整个数据库   -->   mysqldump db1 >/backup/db1.20060725   
压缩备份        -->   mysqldump db1 | gzip >/backup/db1.20060725 
分表备份        -->   mysqldump db1 tab1 tab2 >/backup/db1_tab1_tab2.sql 
直接远程备份     -->   mysqladmin -h boa.snake.net create db1 
              -->   mysqldump db1 | mysql -h boa.snake.net db1 

复制备份表      -->   cp tab.*   backup/ 


恢复 
用最新的备份文件重装数据库。如果你用mysqldump产生的文件，将它作为mysql的输入。如果你用直接从数据库拷贝来的文件，将它们直接拷回数据库目录，然而，此时你需要在拷贝文件之前关闭数据库，然后重启它。  





### 3. 彻底卸载（不保留数据）

1、 rpm -qa | grep -i mysql

查找已经安装的mysql.

MySQL-server-5.6.43-1.el6.x86_64 

MySQL-client-5.6.43-1.el6.x86_64

MySQL-devel-5.6.43-1.el6.x86_64

以上三个就是我安装的mysql.

2、 yum -y remove MySQL-*

网上的一般用rpm -e 的命令删除mysql,这样表面上删除了mysql,可是mysql的一些残余程序仍然存在,并且通过第一步的方式也查找不到残余,而yum命令比较强大,可以完全删除mysql.(ps:用rpm删除后再次安装的时候会提示已经安装了,这就是rpm没删除干净的原因)

3、 find / -name mysql

查找mysql的一些目录,把所有出现的目录统统删除.可以使用rm -rf 路径，删除时请注意，一旦删除无法恢复。

4、rm -rf /etc/my.cnf

这个是删除配置文件

5、 rm -rf /root/.mysql_sercret

删除mysql的默认密码,如果不删除,以后安装mysql这个sercret中的默认密码不会变,使用其中的默认密码就可能会报类似Access denied for user 'root@localhost' (using password:yes)的错误.

五步完成之后,这样mysql就全部删除干净了.

# 2. ubuntu18.04

一，彻底删除mysql5.7
一，查看mysql的依赖项：

`dpkg --list | grep mysql`
二，卸载

`sudo apt-get remove mysql-common`
三，卸载（最后的版本数字根据自己具体的版本进行相应的修改）

`sudo apt-get autoremove --purge mysql-server-5.7`
四，清楚残留数据

`dpkg -l|grep ^rc|awk '{print$2}'|sudo xargs dpkg -P`
五，再次查看依赖项

`dpkg --list|grep mysql`
若命令输入之后无反应直接出现命令提示符，则说明依赖项完全删除：（这里我重复输入了两边）



若仍有其他内容，则继续清除剩余依赖项：（这里的命令与上一条清除命令不同）

`sudo apt-get autoremove --purge mysql-apt-config`


最后查看依赖项；无，完全删除；



## 2.1 安装

 `wget https://dev.mysql.com/get/mysql-apt-config_0.8.19-1_all.deb`

`dpkg -i mysql-apt-config_0.8.11-1_all.deb`

`apt-get update`

`apt-get install mysql-server`

参考链接： [Ubuntu18.04安装Mysql8.0_bird of paradise的博客-程序员资料_ubuntu安装mysql8.0 - 程序员资料 (4k8k.xyz)](http://www.4k8k.xyz/article/weixin_43900412/85053662)

`service mysql start`

`mysql -uroot -p`

### 2.1.1 配置

`use mysql`

`update user set host=% where user=root;`

`flush privileges`

### 2.1.2 创建删除授权

·创建用户

```sql
create user 'test1'@'localhost' identified by '密码';
```

Mysql8.0默认采用caching-sha2-password加密，目前好多客户端不支持，可改为mysql_native_password;

```mysql
create user 'test'@'%' identified with mysql_native_password BY '密码';
```

其中localhost指本地才可连接

可以将其换成%指任意ip都能连接

也可以指定ip连接

* 查询用户信息:

```mysql
select host, user, authentication_string, plugin from user;
```



## 修改密码

```pgsql
Alter user 'test1'@'localhost' identified by '新密码';
ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges;
```

## 授权

```mysql
grant all privileges on *.* to 'test1'@'localhost' with grant option;
GRANT select,update,insert,create on *.* TO 'xxh'@'%' WITH GRANT OPTION;

```

with gran option表示该用户可给其它用户赋予权限，但不可能超过该用户已有的权限

比如a用户有select,insert权限，也可给其它用户赋权，但它不可能给其它用户赋delete权限，除了select,insert以外的都不能

这句话可加可不加，视情况而定。

all privileges 可换成select,update,insert,delete,drop,create等操作 如：grant select,insert,update,delete on *.* to 'test1'@'localhost';

第一个*表示通配数据库，可指定新建用户只可操作的数据库

如：grant all privileges on 数据库.* to 'test1'@'localhost';

第二个*表示通配表，可指定新建用户只可操作的数据库下的某个表

如：grant all privileges on 数据库.指定表名 to 'test1'@'localhost';

## 刷新权限

```sql
flush privileges;
```

## 查看用户授权信息

```ada
show grants for 'test1'@'localhost';
```

## 撤销权限

```pgsql
revoke all privileges on *.* from 'test1'@'localhost';
```

用户有什么权限就撤什么权限

## 删除用户

```n1ql
drop user 'test1'@'localhost';
```

## 3. 搭建集群

[(3条消息) MySQL8.0高级应用———集群模式配置（主从、双主、MHA高可用模式配置）_richie696的专栏-CSDN博客](https://blog.csdn.net/richie696/article/details/114261284?utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-1.no_search_link)

参考这个博客。



注意 主从每个机子的server-id 必须不一样

```mysql
change master to master_host='172.16.6.221',master_port=3306,master_user='xxh',master_password='',master_log_file='mysql-bin.000001',master_log_pos=156;
```

