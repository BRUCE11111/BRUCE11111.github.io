---
title: linux基础
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-10 11:28:08
password:
summary:
tags: linux
categories: 基础
---

## 1.命令1

### 2. 查看当前路径

```shell
pwd
-L 
如果 PWD 环境变量包含了不包含文件名 .（点表示当前目录）或 ..（点点表示父目录）的当前目录的绝对路径名，则显示 PWD 环境变量的值。否则，-L 标志与 -P 标志一样运行。 
-P 
显示当前目录的绝对路径名。与 -P 标志一起显示的绝对路径不包含在路径名的绝对路径中涉及到符号链接类型的文件的名称。
```

### 3. 修改环境变量

使用export命令直接修改PATH的值，配置MySQL进入环境变量的方法:

```shell
export PATH=/home/uusama/mysql/bin:$PATH
#或者把PATH放在前面
export PATH=$PATH:/home/uusama/mysql/bin
```

注意事项：

- 生效时间：立即生效
- 生效期限：当前终端有效，窗口关闭后无效
- 生效范围：仅对当前用户有效
- 配置的环境变量中不要忘了加上原来的配置，即$PATH部分，避免覆盖原来配置

#### Linux环境变量配置方法二：vim ~/.bashrc

通过修改用户目录下的~/.bashrc文件进行配置：

```shell
vim ~/.bashrc
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin
```

注意事项：

- 生效时间：使用相同的用户打开新的终端时生效，或者手动source ~/.bashrc生效
- 生效期限：永久有效
- 生效范围：仅对当前用户有效
- 如果有后续的环境变量加载文件覆盖了PATH定义，则可能不生效

#### Linux环境变量配置方法三：vim ~/.bash_profile

和修改~/.bashrc文件类似，也是要在文件最后加上新的路径即可：

```shell
vim ~/.bash_profile
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin
```

注意事项：

- 生效时间：使用相同的用户打开新的终端时生效，或者手动source ~/.bash_profile生效
- 生效期限：永久有效
- 生效范围：仅对当前用户有效
- 如果没有~/.bash_profile文件，则可以编辑~/.profile文件或者新建一个

#### Linux环境变量配置方法四：vim /etc/bashrc

该方法是修改系统配置，需要管理员权限（如root）或者对该文件的写入权限：

```shell
# 如果/etc/bashrc文件不可编辑，需要修改为可编辑
chmod -v u+w /etc/bashrc
vim /etc/bashrc
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin
```

注意事项：

- 生效时间：新开终端生效，或者手动source /etc/bashrc生效
- 生效期限：永久有效
- 生效范围：对所有用户有效

#### Linux环境变量配置方法五：vim /etc/profile

该方法修改系统配置，需要管理员权限或者对该文件的写入权限，和vim /etc/bashrc类似：

```shell
# 如果/etc/profile文件不可编辑，需要修改为可编辑
chmod -v u+w /etc/profile
vim /etc/profile
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin
```

注意事项：

- 生效时间：新开终端生效，或者手动source /etc/profile生效
- 生效期限：永久有效
- 生效范围：对所有用户有效

#### Linux环境变量配置方法六：vim /etc/environment

该方法是修改系统环境配置文件，需要管理员权限或者对该文件的写入权限：

```shell
# 如果/etc/bashrc文件不可编辑，需要修改为可编辑
chmod -v u+w /etc/environment
vim /etc/profile
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin
```

注意事项：

- 生效时间：新开终端生效，或者手动source /etc/environment生效
- 生效期限：永久有效
- 生效范围：对所有用户有效

#### 一些小技巧

可以自定义一个环境变量文件，比如在某个项目下定义uusama.profile，在这个文件中使用export定义一系列变量，然后在~/.profile文件后面加上：sourc uusama.profile，这样你每次登陆都可以在Shell脚本中使用自己定义的一系列变量。

也可以使用alias命令定义一些命令的别名，比如alias rm="rm -i"（双引号必须），并把这个代码加入到~/.profile中，这样你每次使用rm命令的时候，都相当于使用rm -i命令，非常方便。

#### 1.僵尸进程

当子进程退出时，父进程没有调用wait函数或者waitpid()函数等待子进程结束，又没有显式忽略SIGCHLD信号，那么它将一直保持在僵尸状态，如果这时父进程结束了，init进程会自动接收这个子进程，为它收尸，但如果父进程是一个循环，不会结束，那么子进程就会一直保持僵死状态。

进程状态：

- Z 僵尸
- S 休眠
- D 不可中断的休眠
- R 运行
- T 停止时跟踪

:icecream: 例子1：

* 查看僵尸进程：`ps -A -o stat,ppid,pid,cmd | grep -e '^[Zz]'`

命令注解：

-A 参数列出所有进程

-o 自定义输出字段 我们设定显示字段为 stat（状态）, ppid（进程父id）, pid(进程id)，cmd（命令）这四个参数

* 杀死僵尸进程：

`ps -A -o stat,ppid,pid,cmd | grep -e '^[Zz]' | awk '{print $2}' | xargs kill -9`

查找僵死进程，然后将父进程杀死。

### ubuntu 查询dns

`nmcli dev show`

### 固定ip

#### ubuntu20.04 （18 也行）

禁用dhcp使用静态ip注意事项

假设要把ip地址设为 192.168.1.38 子网掩码24位即255.255.255.0
首先确认网卡名称 使用ifconfig命令:

```dns
`eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    inet 192.168.1.39  netmask 255.255.255.0  broadcast 192.168.1.255`
```

网卡名称 eno1 目前的ip地址是192.168.1.39,子网掩码24位

确认网关地址:使用 route -n 命令

```accesslog
`Destination     Gateway         Genmask        Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 eno1`
```

默认网关 192.168.1.1

最后编辑 sudo vim /etc/netplan/01-network-manager-all.yaml 填入:

```yaml
`network:
  version: 2
  renderer: NetworkManager
  ethernets:
    eno1:   # 网卡名称
      dhcp4: no     # 关闭dhcp
      dhcp6: no
      addresses: [192.168.1.38/24]  # 静态ip
      gateway4: 192.168.1.1     # 网关
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]`# dns
        
```

运行 `sudo netplan apply`

## 4. linux各个目录的作用

`bin` 存放系统命令的目录，普通用户和超级用户都可以执行。/bin目录下的命令单用户模式下也可以执行。

`sbin`保存系统环境设置相关的命令，只有超级用户可以使用这些命令进行系统环境设置，但是有些命令允许普通用户查看。

`usr/bin`存放系统命令的目录，普通用户和超级用户都可以执行。这些命令和系统启动无关，在单用户模式下不能执行。

`boot`系统启动目录，保存系统启动相关的文件，如内核文件和启动引导(grub)程序文件。

`dev` 设备文件保存位置。Linux中所有内容以文件形式保存，包括硬件。这个目录就是用来保存所有硬件设备文件的。

`/etc/` 配置文件保存位置。系统内所有采用默认安装方式的服务的配置文件全部保存在这个目录中，如用户账户和密码，服务的启动脚本，常用服务的配置文件。

`home`建立每个用户时，每个用户都需要一个默认位置，这个位置就是用户家目录。所有普通用户的家目录就是在/home下建立一个和用户名相同的目录。如用户usr1的家目录就是/home/user1。

`lib`系统调用的函数库位置

`/lost+found/`当系统意外崩溃和机器意外关机，而产生一些文件碎片放在这里。当系统启动的过程中fsck工具会检查这里，并修复已经损坏的文件系统。这个目录只在每个分区中出现，例如/lost+fount就是根分区的备份恢复目录。/boot/lost+found 就是/boot分区的备份恢复目录。

`/mnt`挂载目录，早期linux中只有这一个挂载目录，并没有细分。现在这个目录系统建议挂载额外设备。

`mnt`挂在U盘或者移动硬盘。

`/opt`第三方安装的软件保存位置。 第三方现在习惯放到了/usr/local下。

`proc` 虚拟文件系统，该目录中的数据并不保存在硬盘中，而是保存在内存当中。

`sys` 虚拟文件系统。和/proc目录相似，都是保存在内存当中的，主要是保存于内核相关信息的。

`usr` 系统软件资源目录。 unix software resource 的缩写，不是存放用户数据，而是存放系统软件资源的目录。系统中安装的大多数软件都存放在这里。

`var`动态数据（比如日志）保存的位置。缓存、日志以及软件运行所产生的文件。

### 4.1 文件处理

#### 4.1.1 ls

`ls -la /etc` 命令 [-选项] [参数]

ls -l (long 长格式显示)

显示的格式如下：

​							用户	用户组 文件大小（字节） 时间 文件名

`-rw------- 1 root root 1205 3月 3 08:10 anaconda-ks.cfg`

\- 文件类型（\- 二进制文件 d目录 l 软链接文件）

其后分别是 u g o

u所有者 g所属组 o其他人

r读 w写 x执行（最高权限）

![image-20211107134330379](linux%E5%9F%BA%E7%A1%80/image-20211107134330379.png)

#### 4.1.2 mkdir

`mkdir -p` 递归创建目录。

`mkdir -p /tmp/japan/boduo`

`mkdir /tmp/Japan/longze  /tmp/Japan/cangjing`

#### 4.1.3 cd (change directory) 

打开目录

#### 4.1.4 rmdir

删除目录。

#### 4.1.5 cp (copy)

cp -rp [原文件或目录] [目标目录]

-r 复制目录

-p 保留文件属性

#### 4.1.6 touch

创建空文件

#### 4.1.7 cat

`cat -n` 显示行号

tac 反着显示

#### 4.1.8 more

分页显示文件的命令。

#### 4.1.9 less

less命令可以向上翻页。

浏览状态下，某个关键词位置。/关键词 就会找到，然后n键可以往下走。

#### 4.1.10 head

只看文件前几个部分。

`head -n 10 文件名` 看前10行

#### 4.1.11 tail

查看末尾的几行，同head。

#### 4.1.12 ln

`ln -s /etc/issue /tmp/issue.soft`

创建/etc/issue的软连接/tmp/issue.soft

硬链接：不能指向目录、不能跨分区。

#### 4.1.13 locate

find在某一个文件或者分区硬盘上全部遍历查找。

locate是直接找资料库。系统有一个维护的文件资料库，

如果使用`locate locate`寻找，那么就会发现打印出来的

![image-20211013124132522](D:\1\blog\source\_posts\linux基础\image-20211013124132522.png)

白色就是资料库。``但是如果新建立的文件，没有被收录到这个资料库，那么这个文件是找不到的``。

我们需要更新资料库，`updatedb` （除了tmp临时文件夹下的文件无法更新）

这个时候就会检索到新建立的文件。

* 不区分大小写 -i

#### 4.1.14 which

搜索命令所在的目录及别名信息

#### 4.1.15 whereis

和which功能一样，区别在于还列出了帮助文档。

#### 4.1.16 grep

文件内容中进行搜索。

执行权限 ： 所有用户。

`grep -iv [执行字串][文件]` `-i 不区分大小写 -v 排除指定字串`

在文件内，某个字串所在的行找出来

![image-20211013130905308](D:\1\blog\source\_posts\linux基础\image-20211013130905308.png)

`grep -v ^# /etc/..file` 找到所有并排除以 # 开头的行

#### 4.1.17 find

`find /etc -name init` 在etc文件夹下寻找名字为init的文件，这个一般是精准搜索，全匹配。

如果要模糊搜索，可以：`find /etc -name *init*` 使用通配符。

`find /etc/ -name init???` 匹配三个字符 问好匹配单个字符

`find / -size +204800` 在根目录下查找大于100MB的文件。 +n 大于 -n 小于 n 等于。 一个数据块0.5k。 然后 100MB = 102400KB * 2 = 204800

`find /home -user shenchao` 在home文件夹下找到所有者为shenchao的文件。

`find /etc -cmin -5` 在/etc目录下查找5分钟内（注意-号，是小于5分钟）被修改过的属性的文件和目录。

-amin 访问时间 access

-cmin 文件属性 change

-mmin 文件内容 modify



`find /etc -size + 163840 -a -size -204800` 在etc下查找大于80MB小于100MB的文件。

-a 两个文件同时满足 -o 两个条件满足任何一个即可。

-type f 文件 d目录 l软连接文件

`find /etc -name inittab -exec ls -l {}\;` 在etc下查找inittab文件并显示其详细信息

-exec/-ok 命令{}\ ，对搜索结果执行操作。

-inum i节点， 根据i节点查找。

`find -inum 31531 -exec rm {} \` 删除i节点是31531的文件。

#### 4.1.18 history

`history [选项] [历史命令保存文件]`

-c 清空历史命令

-w 把缓存中的历史命令写入历史命令保存文件 ~/.bash_history

历史命令默认保存1000条，在`/etc/profile`文件中修改。

> 执行历史命令
>
> 1. `!n` 执行第n条历史命令。
> 2. `!!` 重复执行上一条命令。
> 3. `!字串` 重复执行最后一条以字串开头的命令。

### 4.2 帮助命令

#### 4.2.1 man

命令和配置文件的帮助信息。

`man ls` 查看ls命令的帮助信息

1 是命令的帮助 5 是配置文件的帮助。

#### 4.2.2 whatis 

简短地得到命令地信息

#### 4.2.3 apropos

查看配置文件的信息 `apropos services` 

#### 4.2.4 help

有一部分是shell内置命令。找不到它存放的路径。需要使用help获取内置命令的帮助信息。

### 4.3 用户管理

#### 4.3.1 useradd

添加用户命令。

#### 4.3.2 passwd

更改密码 `passwd xxh`

#### 4.3.3 who

![image-20211014141358511](D:\1\blog\source\_posts\linux基础\image-20211014141358511.png)

### 4.3.4 w

![image-20211014141459636](D:\1\blog\source\_posts\linux基础\image-20211014141459636.png)

IDLE 空闲时间

JCPU 累计占用cpu时间

PCPU 当前用户占用cpu的时间。

WHAT 当前执行的命令。

#### 4.3.5 uptime

总共运行时间。

#### 4.3.6 chmod

`chmod [{ugoa}{+=}{rwx}] [文件或目录][mode=421][文件或目录] -R 递归修改`

### 4.3.7 踢出用户

\#who

USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
root pts/0 192.168.153.72 14:52 10.00s 0.00s 0.00s w
root pts/1 192.168.153.72 14:52 0.00s 0.00s 0.00s -bash

方法一：

`pkill -kill -t pts/1`

方法二：

`fuser -k /dev/pts/1`

你也可以给他发送关闭信息然后关闭

`echo "你被管理员踢出了" > /dev/pts/1 && fuser -k /dev/pts/1`



### 4.4 压缩

* 压缩格式 gzip

.gz 对应命令 `gzip`  解压缩 `gunzip`

`gzip 文件名` gzip 只能压缩文件，不能压缩目录。

压缩完就只剩下压缩包了，源文件就不见了。

* tar

`tar -cvfz [-zcf][压缩后文件名][目录]`

-c 打包

-v 显示详细信息

-f 指定文件名

-z 打包同时压缩 或者 解压缩

-x 解包

-j bzip2 的格式

* zip

`zip 选项[-r] [压缩后的文件名] [文件或目录]`  -r压缩目录 

比gzip压缩效果差

* bzip2 zip的升级版本

`bzip2 选项[-k] [文件]` 压缩后保留原文件 

压缩比很厉害

### 4.5 网络命令

#### 4.5.1 write

命令所在路径：`/usr/bin/write`

语法： write <用户名>

功能： 给在线用户发信息，以ctrl + D 保存结束

#### 4.5.2 wall 

广播所有在线用户发送信息

`wall message`

#### 4.5.3 ping

测试网络的连通性

`ping ip`

`ping -c 3 ip` 打印3次信息

#### 4.5.4 ifconfig

![image-20211023205703132](D:\1\blog\source\_posts\linux基础\image-20211023205703132.png)



#### 4.5.5 mail

查看发送电子邮件

`mail 用户名` 

接收 ： mail 啥都不用加，就可以看了

#### 4.5.6 last

统计计算机所有用户的登录和重启时间

#### 4.5.7 lastlog

`lastlog -u 用户名`

不加-u 可以打印全部用户

查看所有用户 最近登录的信息

#### 4.5.8 traceroute

`traceroute 网络地址`

#### 4.5.9 netstat

查询网络状态相关信息

-t tcp协议 

-u udp

-l 监听

-r 路由 计算机网关

-n 显示IP地址和端口号

一般就是写 -tuln 就满足了大部分场景

#### 4.5.10 setup

redhat系列系统才有的。

网络配置。



### 4.6 关机重启命令

* shutdown

-c： 取消前一个关机命令

-h: 关机

-r：重启

尽量使用shutdown命令来进行关机重启。因为它会将服务正常关闭。

其他的关机命令

* halt 
* poweroff // 直接断电
* init 0

* reboot 重新启动
* init 6 重新启动

**系统运行级别 **7个运行级别

* 0 关机
* 1 单用户
* 2 不完全多用户，不含NFS服务
* 3 完全多用户
* 4 未分配
* 5 图形界面
* 6 重启

runlevel 查看运行级别

* logout 退出登录

### 4.7 文本编辑器 vim

![image-20211027172327338](linux%E5%9F%BA%E7%A1%80/image-20211027172327338.png)



![image-20211027190046265](linux%E5%9F%BA%E7%A1%80/image-20211027190046265.png)

![image-20211027190422338](linux%E5%9F%BA%E7%A1%80/image-20211027190422338.png)

![image-20211027191046576](linux%E5%9F%BA%E7%A1%80/image-20211027191046576.png)



#### 4.7.1 常用操作

![image-20211029170805479](linux%E5%9F%BA%E7%A1%80/image-20211029170805479.png)

#### 4.7.2 常用命令

复制和剪切 yy-p 复制到下面 dd-p 剪切到上面

替换和恢复 r/R u

#### 4.7.3 技巧

编辑模式中：

`:r /etc/issue` 将/etc/issue 的文件内容导入到当前光标所在的位置。

`:! 系统命令` 不退出vim中执行命令

`map ^P I#<ESC>` 触发命令

^ 表示行首。

![image-20211029194326440](linux%E5%9F%BA%E7%A1%80/image-20211029194326440.png)

* 配置默认的vim文件，每个用户对应的目录下进行配置：/home/xxh/.vimrc



## 5. RPM软件包

* 源代码包
* 二进制包
  * RPM包和系统默认包

### 5.1 RPM包

#### 5.1.1 RPM包命名规则

![image-20211030193430610](linux%E5%9F%BA%E7%A1%80/image-20211030193430610.png)

#### 5.1.2 RPM包的依赖性

![image-20211030212353633](linux%E5%9F%BA%E7%A1%80/image-20211030212353633.png)

#### 5.1.4 rpm安装

`rmp -ivh 包全名` -i(install) 安装 

-v（verbose）显示详细信息

-h (hash) 显示进度

--nodeps 不检测依赖性

#### 5.1.5 rpm卸载

`rpm -e 包名` 

-e 卸载

--nodeps 不检查依赖性



`rpm -q 包名` 查询是否安装 -q query 的意思

`rpm -qa` 查询素偶又安装的RPM包

`rpm -qi 包名` 查询包信息

`rpm -qip 包名` -p 查询未安装包的信息

`rpm -qR 包名` 查询软件包的依赖性

`rpm -qf 系统文件名` 查询系统文件属于哪个软件包

`rpm -V 包名` 校验指定RPM包中的文件（verify)

![image-20211031215839628](linux%E5%9F%BA%E7%A1%80/image-20211031215839628.png)

![image-20211101152754501](linux%E5%9F%BA%E7%A1%80/image-20211101152754501.png)

#### 5.1.6 RPM包中文件提取

`rpm2cpio 包全名 | \` 

cpio -idv .文件绝对路径

rpm2cpio # 将rpm包转换为cpio格式的命令

cpio 是一个标准工具，它用于创建软件档案文件和从档案文件中提取文件。

`cpio 选项 < [文件|设备] ` -i: copy-in模式，还原 

![image-20211101154530780](linux%E5%9F%BA%E7%A1%80/image-20211101154530780.png)

### 5.2 yum在线管理

yum在线管理解决包的依赖性。

`yum list` 查询所有可用软件包列表

`yum search 关键字` 搜索服务器上所有和关键字相关的包

`yum -y update 包名` yum 自动升级包 如果不写包名，**会升级所有的包，并且会升级linux内核**。

`yum -y remove 包名` yum卸载，记得按顺序卸载，先卸载a，否则先卸载c的话，会把依赖于c的所有包全部卸载掉。很容易造成系统故障。

![image-20211102111108482](linux%E5%9F%BA%E7%A1%80/image-20211102111108482.png)

### 5.3 源码包和RPM包的区别

安装之前：概念上的区别

安装之后：安装位置不同

![image-20211102134500800](linux%E5%9F%BA%E7%A1%80/image-20211102134500800.png)

![image-20211102134801312](linux%E5%9F%BA%E7%A1%80/image-20211102134801312.png)

![image-20211102140053616](linux%E5%9F%BA%E7%A1%80/image-20211102140053616.png)

![image-20211102142647407](linux%E5%9F%BA%E7%A1%80/image-20211102142647407.png)



* `./configure` 软件配置与检查



![image-20211102143533533](linux%E5%9F%BA%E7%A1%80/image-20211102143533533.png)

## 6. 用户管理

### 6.1 用户管理配置文件

#### 6.1.1 /etc/passwd

字段1： 用户名称

字段2：密码标志 密码其实放到了shadow文件中。因为用户存放了用户信息，所有用户都可以看到信息，所以放到了权限更低的shadow。shadow当中的密码还是密文，也是比较安全的。x表示这个用户是有密码的。

字段3：UID（用户ID） 0：超级用户。 1-499：系统用户。500-65535：普通用户。

字段4：GID （用户初始组ID）

字段5：用户说明。

字段6：家目录。普通用户/home/用户名。 超级用户:/root/

字段7：登录之后的shell。

#### 6.1.2 /etc/shadow

字段1：用户名

字段2：加密密码，SHA512散列加密算法。 如果密码位是“!!”或"*"代表没有密码，不能登录。

字段3：密码最后一次修改的日期。

字段4：两次密码的修改间隔时间（和3字段相比）。

字段5：密码有效期（和3字段相比）

字段6：密码修改到期前的警告天数（和第5字段相比）。

#### 6.1.3 /etc/group

字段1：组名。

字段2：组密码标志。

字段3：GID。

字段4：组中附加用户。

#### 6.1.4 /etc/gshadow

字段1：组名。

字段2：组密码。 不推荐使用组密码。

字段3：组管理员用户名。

字段4：组中附加用户。

### 6.2 useradd

普通用户：/home/用户名， 所有者和所属组都是此用户，权限是700。

超级用户：/root/ ，所有者和所属组都是root用户，权限是550。

`useradd [选项] 用户名`

-u UID： 手工指定用户的UID号

-d 家目录： 手工指定用户的家目录

-c 用户说明：手工指定用户的说明

-g 组名：手工指定用户的初始组

-G 组名：指定用户的附加组

-s shell：手工指定用户的登录shell。默认是/bin/bash

* /etc/default/useradd

GROUP=100 #用户默认组

HOME=/home #用户家目录

INACATIVE=-1 #密码过期宽限天数（shadow文件7字段）

EXPIRE= #密码失效时间

SHELL=/bin/bash

SKEL=/etc/skel

CREATE_MAIL_SPOOL=yes #是否建立邮箱

* /etc/login.defs

PASS_MAX_DAYS 99999 # 密码有效期（5）

PASS_MIN_DAYS 0 # 密码修改间隔（4）

PASS_MIN_LEN 5 #密码最小5位（PAM） 现在PAM管理，其实是最小8位

PASS_WARN_AGE 7 # 密码到期警告（6）

UID_MIN 500 # 最小和最大UID范围

GID_MAX 60000

ENCRYPT_METHOD SHA512 #加密模式

### 6.3 passwd

修改用户密码。

pass [选项] 用户名

-S 查询用户密码的密码状态。仅root用户可用。

-l 暂时锁定用户。仅root用户可用。

-u 解锁用户，仅root用户可用。

--stdin 可以通过管道符输出的数据作为用户的密码。

### 6.4 usermod和chage

修改用户信息和修改用户密码状态。

* usermod [选项] 用户名

-u UID: 修改用户的UID号

-c 用户说明： 修改用户的说明信息

-G 组名： 修改用户的附加组。

-L： 临时锁定用户（Lock）。

-U： 解锁用户锁定（Unlock）。

* chage [选项] 用户名

-l: 列出用户的详细密码状态

-d 日期： 修改密码最后一次更改日期（shadow3字段）

-m 天数： 两次密码修改间隔（4字段）

-M 天数： 密码有效期（5字段）

-W 天数：密码有效期（5字段）

-I 天数：密码过后宽限天数（7字段）

-E 日期：账号失效时间（8字段）

* userdel [-r] 用户名

-r 删除用户的同时删除用户家目录

![image-20211106161116574](linux%E5%9F%BA%E7%A1%80/image-20211106161116574.png)

### 6.5 用户组管理命令

* 添加组`groupadd [选项] 组名`

-g GID: 指定组ID

* 修改组 groupmod [选项] 组名

-g GID: 修改组ID

-n 新组名： 修改组名

* groupmod -n testgrp group1

把组名group1修改为testgrp

* 删除组 groupdel 组名

* gpasswd 选项 组名

-a 用户名： 把用户加入组

-d 用户名： 把用户从组中删除

`gpasswd -a user1 root` 把user1加入root组。 

或者直接在/etc/group文件中最后加入用户名。

## 7. 权限管理

### 7.1 ACL权限

身份不足问题，因为身份只有所有者、所属组和其他人。

ACL和windows比较接近，直接给这个用户分配权限，不再考虑所有者、所属组啥的。

* 分区必须支持ACL权限

`dumpe2fs -h /dev/sda3` dumpe2fs命令是查询指定分区详细文件系统信息的命令选项：-h 仅显示超级块中信息，而不显示磁盘块组的详细信息。

![image-20211107135443259](linux%E5%9F%BA%E7%A1%80/image-20211107135443259.png)

上面这个图片的信息表示支持ACL。

如果不支持ACL，需要开启，开启方式如下：

`mount -o remount, acl /` 重新挂载根分区，并挂载加入acl权限。这个是临时生效的，若想要永久生效，需要：

`vi /etc/fstab` 这个文件是系统开机自动挂载文件。

![image-20211107135936299](linux%E5%9F%BA%E7%A1%80/image-20211107135936299.png)

修改完毕后，重启或者重新挂载文件 `mount -o remount /`。

### 7.2 ACL查看与设定

* `getfacle 文件名` 查看acl权限

* setfacl 选项 文件名
  * -m 设定ACL权限
  * -x 删除指定的ACL权限
  * -b 删除所有的ACL权限
  * -d 设定默认ACL权限
  * -k 删除默认ACL权限
  * -R 递归设定ACL权限

:icecream: 例子： 给用户`st` 读和执行`/project`目录的权限：

`setfacl -m u:st:rx /project/` 加完权限之后，可以发现`ll`下的权限中多了一个`+`号。说名有其他权限。

![image-20211107141712388](linux%E5%9F%BA%E7%A1%80/image-20211107141712388.png)

查看这个`+`是什么权限，使用命令： `getfacl /project` 

![image-20211107141922160](linux%E5%9F%BA%E7%A1%80/image-20211107141922160.png)

### 7.3 最大有效权限和删除

mask是用来指定最大有效权限的。如果我给用户赋予了ACL权限，需要和mask的权限“相与”才能得到用户的真正权限。

![image-20211107143407319](linux%E5%9F%BA%E7%A1%80/image-20211107143407319.png)

* `setfacl -m m:rx /project/` 最大权限只有读和执行。

* `setfacl -b 文件名` 把文件或目录的所有ACL权限删除
* `setfacl -x g:tgroup2 /project/` 指定删除某一个用户或者组的权限

### 7.4 默认ACL权限和递归ACL权限

#### 7.4.1 递归ACL权限

父目录设定的ACL权限，子目录和子文件也都会有一样的ACL权限。

* `setfacl -m u:用户名:权限 -R 文件名` 其实就是-R参数。针对现有的文件赋值ACL权限。

#### 7.4.2 默认ACL权限

如果父目录设定了默认ACL权限，那么父目录中所有新建的子文件都会继承父目录的ACL权限。

* `setfacl -m d:u:用户名:权限 文件名` 针对未来的文件赋值ACL权限。

### 7.5 文件特殊权限

#### 7.5.1 文件特殊权限SetUID 4

* 只有执行的二进制程序才能设定SUID权限。

* 命令执行者要对该程序拥有x（执行）权限。

* 命令执行者在执行该程序时获得该程序文件属主的身份。

* SetUID权限只在该程序执行过程中有效，也就是身份改变只在程序执行过程中有效。

:shaved_ice: 例子：

`passwd` 命令拥有`SetUID`权限，所以可以修改自己的密码。

![image-20211112123708042](linux%E5%9F%BA%E7%A1%80/image-20211112123708042.png)

`cat`命令没有SetUID权限，所以无法查看`/etc/shadow`文件内容。

![image-20211112124519400](linux%E5%9F%BA%E7%A1%80/image-20211112124519400.png)

设定SetUID的方法：

* chmod 4755 文件名
* chmod u+s 文件名

取消SetUID的方法：

* chmod 755 文件名
* chmod u-s 文件名

4 表示SUID。

这个权限很危险，在使用时必须要谨慎。比如你把`vi`这个命令的可执行文件赋予了setuid权限，那么以后任何一个用户都可以相当于root用户来保存和修改任何一个文件！十分危险！

`find /bin -perm /u=s`  查看/bin目录下的带有s权限的文件。

:tired_face: 注意，如果`ll`后权限中有S，说明赋予属性错误。

#### 7.5.2 SetGID 2

* 只有二进制文件才能设置`SGID`权限
* 命令执行者要对该程序拥有x（执行）权限

* 命令执行在执行程序时，组身份升级为该程序文件的属组。
* SetGID权限同样只在该程序执行过程中有效，也就是说组身份改变只在程序执行过程中有效。

**SetGID对目录的作用：**

普通用户必须对此目录拥有r和x权限，才能进入此目录。

普通用户在此目录中的有效组会变成此目录的属组。

若普通用户对此目录拥有w权限时，新建的文件的默认属组是这个目录的属组。

:icecream: 例子：![image-20211112213029883](linux%E5%9F%BA%E7%A1%80/image-20211112213029883.png)



#### 7.5.3 Sticky BIT 文件特殊权限 1

* 粘着位目前只对目录有效
* 普通用户对该目录拥有w和x权限，即普通用户可以在此目录拥有写入权限。
* 如果没有黏着位，因为普通用户拥有w权限，所以可以删除此目录下所有文件，包括其他用户建立的文件。一旦赋予了黏着位，除了root可以删除所有文件，普通用户就算拥有w权限，也只能删除自己建立的文件，但是不能删除其他用户建立的文件。

设置黏着位 

* chmod 1755 目录名
* chmod o+t 目录名

取消黏着位

* chmod 777 目录名
* chmod o-t 目录名

:icecream: 例子：

![image-20211112213233069](linux%E5%9F%BA%E7%A1%80/image-20211112213233069.png)

### 7.6 chattr权限

`chattr [+-=] [选项] 文件或目录名`

`+:增加权限`

`-:删除权限`

`=:等于某权限`

改变文件属性

`i`: 如果对文件设置i属性，那么不允许对文件进行删除、改名，也不能添加和修改数据。如果对目录设`i`属性，那么只能修改目录下文件的数据，但不允许建立和删除文件。

`a`：如果对文件设置a属性，那么只能在文件中增加数据，但是不能删除和修改数据。如果对目录设置a属性，那么只允许在目录中建立和修改文件，但是不允许删除。

**lsattr** 选项 文件名 （查看权限）

* -a 显示所有文件和目录。
* -d 若目标是目录，仅列出目录本身的属性，而不是子文件的。

:icecream: `chattr +i abc` 添加`i`属性。

### 7.7 sudo权限

* root把本来只能超级用户执行的命令赋予普通用户执行。
* sudo的操作对象是系统命令。

![image-20211113144628539](linux%E5%9F%BA%E7%A1%80/image-20211113144628539.png)

:icecream:例子：

`sc ALL=/sbin/shudown -r now` 写的很细的命令，这一条sc用户就只能使用带着-r参数的命令。

## 8. 分区

### 8.1 分区类型

![image-20211113151814602](linux%E5%9F%BA%E7%A1%80/image-20211113151814602.png)

:icecream:例子：

![image-20211113151824492](linux%E5%9F%BA%E7%A1%80/image-20211113151824492.png)

![image-20211113151937225](linux%E5%9F%BA%E7%A1%80/image-20211113151937225.png)

![image-20211113152050859](linux%E5%9F%BA%E7%A1%80/image-20211113152050859.png)

1 2 3 4 只能分配给主分区或者扩展分区号，逻辑分区只能从5开始。

### 8.2 df命令

`df [选项][挂载点]` 显示文件系统大小和使用情况

选项：

 -a 显示所有的文件系统信息，包括特殊文件系统，如：/proc、/sysfs。

-h 使用习惯单位显示容量，如KB，MB或GB等。

-T 显示文件系统类型。

-m 已MB为单位显示容量。

-k 以KB单位显示容量。默认就是KB为单位。

### 8.3 统计目录或文件大小

`du [选项] [目录或文件名]`

-a 显示每个子文件的磁盘占用量。默认只统计子目录的磁盘占用量

-h 使用习惯单位显示磁盘占用量，如KB，MB或GB等

-s 统计总占用量，而不列出子目录和子文件的占用量。

:satisfied: du和df的区别是，df显示的空间大小一般大于du，因为长时间不重启机子，某些进程所留下的临时文件或删除的文件空间并没有释放。

### 8.4 文件系统修复命令 fsck

`fsck [选项] 分区设备文件名` 

-a：不用显示用户提示，自动修复文件系统。

-y：自动修复。和-a作用一致，不过有些文件系统只支持-y。

### 8.5 磁盘状态命令dumpe2fs

`dumpe2fs 分区设备文件名` 这个命令还可以查询到设备的UUID号。

### 8.6 mount查询和自动挂载

mount命令本身可以查询已经挂载的设备。

`mount [-l]` 查询系统中已经挂载的设备，-l会显示卷标名称。

`mount -a` 依据配置文件`/etc/fstab`的内容，自动挂载。

`mount [-t 文件系统] [-L 卷标名] [-o 特殊选项] 设备文件名 挂载点`

选项： -t 文件系统：加入文件系统类型来指定挂载的类型，可以ext3、ext4、iso9660等文件系统

-L 卷标名： 挂载指定卷标的分区，而不是安装设备文件名挂载

-o 选项特殊：可以指定挂载的额外选项

![image-20211113172923073](linux%E5%9F%BA%E7%A1%80/image-20211113172923073.png)

#### 8.6.1 挂载光盘和U盘

> 建立挂载点
>
> `mkdir /mnt/cdrom/`
>
> 挂载光盘
>
> `mount -t iso9660 /dev/cdrom /mnt/cdrom/` 把设备文件挂载到/mnt的这个目录下
>
> `mount /dev/sr0 /mnt/cdrom` 把设备文件挂载到/mnt的这个目录下

> 卸载命令
>
> `umount 设备文件名或挂载点`
>
> `umount /mnt/cdrom` 



#### 8.6.2 挂载u盘

`fdisk -l` 查看分区

`mount -t vfat /dev/sdb1 /mnt/usb/`

:angry: linux默认是不支持NTFS文件系统的。

#### 8.6.3 支持NTFS文件系统

* 重新编译内核的方式，不推荐 :happy: 。
* 利用第三方软件: ntfs-3g



> ntfs-3g
>
> > `tar -zxvf ntfs-3g_ntfsprogs-2013.1.13.tgz`
> >
> > `cd ntfs-3gxxx`
> >
> > `./configure`
> >
> > `make`
> >
> > `make install`
> >
> > `mount -t ntfs-3g 分区设备文件名 挂载点`

### 8.7 fdisk分区

查看硬盘：

`fdisk -l`

`fdisk /dev/sdb`

![image-20211114195657353](linux%E5%9F%BA%E7%A1%80/image-20211114195657353.png)

重新读取分区表信息： `partprobe`

#### 8.7.1 格式化分区

`mkfs -t ext4 /dev/dsb1`

### 8.8 分区自动挂载和fstab文件修复

> 自动挂载
>
> > 写入`/etc/fstab` 文件。 一定要写对，如果写不对可能导致系统崩溃。
> >
> > :happy: `fstab` 文件内容：
> >
> > ![image-20211117213822198](linux%E5%9F%BA%E7%A1%80/image-20211117213822198.png)
>
> :cry: `fstab` <font color = 'red'>内容如果改错了咋办？</font>
>
> ![image-20211117220201017](linux%E5%9F%BA%E7%A1%80/image-20211117220201017.png)
>
> 使用mount 重新挂载才能修改`fstab` 文件。

## 9.SHELL

![image-20211117220524694](linux%E5%9F%BA%E7%A1%80/image-20211117220524694.png)

SHELL 是一种编程语言，是解释执行的脚本语言，在SHELL中可以直接调用LINUX系统命令。

SHELL有两种语法类型：Bourne和C。两种互不兼容。Bourne主要支持sh、ksh、<font color = "red">Bash</font>、psh、zsh等。C家族主要包括csh和tcsh。

现在linux基本都是用Bash SEHLL。

> linux中支持的SEHLL： 写在文件 `/etc/shells/`

### 9.1 脚本执行方式

#### 9.1.1 echo 

> echo [选项] [输出内容] <font color = "red">输出的内容有空格，必须加双引号或者单引号</font>
>
> > -e 支持反斜线控制的字符转换
> >
> > ![image-20211118111309140](linux%E5%9F%BA%E7%A1%80/image-20211118111309140.png)
> >
> > `echo -e "\e[1;31m abcd \e[0m"` 输出红色字体的`abcd`
>
> > `#!/bin/shell` 开头标记，标注以下程序是SHELL程序。简单的shell程序如果没有标记，不影响。但是SHELL中如果嵌套了其他语句就会报错。
> >
> > 编写完毕一个shell脚本后，有两种执行方式：
> >
> > > 1. 赋予执行权限
> > >
> > >    `chmod 755 xxx.sh`
> > >
> > >    `./xxx.sh` or `sh xxx.sh`
> > >
> > > 2. bash运行
> > >
> > >    `bash xxx.sh`
>
> > 

#### 9.1.2 命令别名

`alia 别名='原命令'` 只能临时生效

`alia` 查询别名 

![image-20211119110453078](linux%E5%9F%BA%E7%A1%80/image-20211119110453078.png)

`vim /root/.bashrc` 别名永久生效。

`unalias 别名` 删除别名。

#### 9.1.3 bash常用快捷键

![image-20211119111227442](linux%E5%9F%BA%E7%A1%80/image-20211119111227442.png)

#### 9.1.4 输入输出重定向

> 输出

![image-20211119111818726](linux%E5%9F%BA%E7%A1%80/image-20211119111818726.png)

![image-20211119112237251](linux%E5%9F%BA%E7%A1%80/image-20211119112237251.png)

输出重定向，不再输出到原始设定的屏幕上，而是文件中。（只有有输出的命令才可以重定向）

![image-20211119113829091](linux%E5%9F%BA%E7%A1%80/image-20211119113829091.png)

> 输入
>
> `wc [选项] [文件名]` 
>
> -c 统计字节数
>
> -w 统计单词数
>
> -l 统计行数
>
> :ice_cream: 例子![image-20211119114853618](linux%E5%9F%BA%E7%A1%80/image-20211119114853618.png) 双`<<` 碰到相同的，之间的内容进行统计。
>
> :ice_cream:例子：![image-20211119114941398](linux%E5%9F%BA%E7%A1%80/image-20211119114941398.png)

#### 9.1.5 多命令顺序执行和管道符

![image-20211119115138032](linux%E5%9F%BA%E7%A1%80/image-20211119115138032.png)

`dd` 命令主要用来进行磁盘复制。

![image-20211119115910670](linux%E5%9F%BA%E7%A1%80/image-20211119115910670.png)

> 管道符
>
> `命令1 | 命令2` 命令1的正确输出作为命令2的操作对象。 命令1必须有正确的输出，否则命令2是不能执行的。
>
> :ice_cream: 例子：![image-20211119121539105](linux%E5%9F%BA%E7%A1%80/image-20211119121539105.png)
>
> 

#### 9.1.6 通配符与其他特殊符号

> 通配符
>
> ![image-20211119122607603](linux%E5%9F%BA%E7%A1%80/image-20211119122607603.png)
>
> :ice_cream: 例子： 
>
> `ls *abc` `ls [^a-z]*`  

> 其他特殊符号
>
> ![image-20211120130200666](linux%E5%9F%BA%E7%A1%80/image-20211120130200666.png)

### 9.2 bash的变量

#### 9.2.1 用户自定义变量

* 不能用数字开头
* 默认字符串类型，如果要进行数值运算，必须指定变量类型为数值型
* 等号两侧不要加空格

:happy: 环境变量名建议大写，便于区分。

#### 9.2.2 环境变量

* 和操作系统环境相关的数据。

> 本地变量
>
> :ice_cream: 例子：`sc=$(name)` 

> 查看变量
>
> :ice_cream: `set` `unset 变量名称`  

> 变量叠加
>
> :ice_cream: `aa=123` `aa="$aa"456` `aa=${aa}789`

> 环境变量设置
>
> `export 变量名=变量值`
>
> 查询变量`env`
>
> 删除变量 `unset 变量名`

> PATH
>
> 系统查找命令的变量
>
> 一般更习惯于变量叠加的方式： `PATH="$PATH":/XX/XX`

![image-20211120160046419](linux%E5%9F%BA%E7%A1%80/image-20211120160046419.png)

#### 9.2.3 位置参数变量

> 几种位置参数变量
>
> ![image-20211120160757024](linux%E5%9F%BA%E7%A1%80/image-20211120160757024.png)

#### 9.2.4 预定义变量

![image-20211120170303489](linux%E5%9F%BA%E7%A1%80/image-20211120170303489.png)

#### 9.2.5 bash的运算符

#### 9.2.5.1 数值运算与运算符

![image-20211120172546934](linux%E5%9F%BA%E7%A1%80/image-20211120172546934.png)

3种将变量声明为数值型的方式：

:ice_cream:例子： `declare -i cc=$aa+$bb`

:ice_cream:例子：`dd=$(expr $aa + $bb)` 注意加号的左右两侧空格必须要加

:ice_cream:例子： `ff=$(($aa+$bb))` 更推荐使用这种两个括号叠加的方式 `$((运算式))` 这里面的空格都不严格，看自己咋写吧

> 运算符优先级 数值越高优先级越高
>
> ![image-20211121134043295](linux%E5%9F%BA%E7%A1%80/image-20211121134043295.png)

####  9.2.5.2 变量测试与内容替换

![image-20211121134333775](linux%E5%9F%BA%E7%A1%80/image-20211121134333775.png)

### 9.3 环境变量配置文件

#### 9.3.1 环境变量配置文件简介

`source 配置文件` 或 `. 配置文件`

环境变量配置文件中主要是定义对系统的操作环境生效的系统默认环境变量，比如PATH、HISTSIZE、PS1、HOSTNAME等默认环境变量中。

#### 9.3.2 环境变量配置文件作用

`/etc/profile`

`/etc/profile.d/*.sh`

`~/.bash_profile`

`~/.bashrc`

`/etc/bashrc`

下图中，上面一行是有登录过程的变量定义文件。

下一行是没有登录过程的变量定义文件，且定义在后面的文件会覆盖前面的文件中的变量。

![image-20211121140539041](linux%E5%9F%BA%E7%A1%80/image-20211121140539041.png)

> `/etc/profile` 作用
>
> ![image-20211121140951883](linux%E5%9F%BA%E7%A1%80/image-20211121140951883.png)



#### 9.3.3 其他配置文件和登录信息

> shell登录信息
>
> > 本地终端欢迎信息 `/etc/issue`
> >
> > ![image-20211121144836187](linux%E5%9F%BA%E7%A1%80/image-20211121144836187.png)
>
> > 远程终端 `/etc/issue.net`
> >
> > 远程终端中不能使用上文中的转义符。
> >
> > 是否显示此欢迎信息，由ssh的配置文件`/etc/ssh/sshd_config`决定，加入"Banner /etc/issue.net"行才能显示（重启ssh）。
>
> > 登陆后的欢迎信息
> >
> > `/etc/motd` 

### 9.4 正则表达式

正则表达式用来在文件中匹配符合条件的字符串，正则是包含匹配。grep、awk、sed等命令可以支持正则表达式。

#### 9.4.1 基础正则表达式

![image-20211121152030768](linux%E5%9F%BA%E7%A1%80/image-20211121152030768.png)

#### 9.4.2 字符截取命令

grep提取行，下面这几个提取列。

> cut 字段提取命令 截断制表符，不是空格
>
> `cut [选项] 文件名`
>
> -f 列号： 提取第几列
>
> -d 分隔符： 按照指定分隔符分割列
>
> :ice_cream: 例子： `cut -f 2,4 student.txt` 提取第二列和第四列
>
> :ice_cream:例子： `cut -d ":" -f 2,4 /etc/passwd` 指定分隔符后提取列



> printf
>
> `printf '输出类型输出格式' 输出内容`
>
> `%ns`： 输出字符串。n是数字指代输出几个字符。
>
> `%ni`: 输出整数。n是数字指代输出几个数字。
>
> `%m.nf`： 输出浮点数。m和n是数字，指代输出的整数位数和小数位数。如`%8.2f`代表共输出8位数，其中2位是小数，6位是整数。
>
> ![image-20211121154304412](linux%E5%9F%BA%E7%A1%80/image-20211121154304412.png)
>
> :ice_cream:例子： `pirntf '%s' $(cat student.txt)`



> awk
>
> awk支持print和printf命令。
>
> print会在每个输出后自动加入一个换行符，printf需要手动加入换行符。
>
> `awk ‘条件1{动作1}条件2{动作2}...’ 文件名`
>
> ![image-20211121155457112](linux%E5%9F%BA%E7%A1%80/image-20211121155457112.png)
>
> :ice_cream:例子： `awk {printf $2 "\t" $6"\n"} student.txt`
>
> BEGIN 动作首先发生。
>
> :ice_cream: `awk 'BEGIN{} {print dd}'`
>
> :ice_cream:例子： `awk '{BEGIN{FS=":"}{print $1}' xxx.txt` FS指定分隔符
>
> END 动作最后发生。
>
> :ice_cream:例子：`awk '{BEGIN{FS=":"}end{print aaaaa}{print $1}' xxx.txt`
>
> 关系运算符 `$6>=88`



> sed
>
> sed是一种几乎包括在所有UNIX平台（包括Linux）的轻量级流编辑器。sed主要是用来将数据进行选取、替换、删除、新增的命令。
>
> sed只能通过命令行输入编辑命令、指定文件名，然后在屏幕上查看输出。sed编辑器没有破坏性，它不会修改文件，除非使用shell重定向来保存输出结果。默认情况下，所有的输出行都被打印到屏幕上。
>
> > :ice_cream:工作过程：
> >
> > sed编辑器逐行处理文件（或输入），并将输出结果发送到屏幕。sed把当前正在处理的行保存在一个临时缓存区中，这个缓存区称为模式空间或临时缓冲。sed处理完模式空间中的行后，就把改行发送到屏幕上。sed把每一行都保存在临时缓存区中，对这个副本进行编辑，所以不会修改或破坏源文件。
> >
> > ![image-20220425231816143](linux%E5%9F%BA%E7%A1%80/image-20220425231816143.png)
>
> 
>
> 
>
> `sed [选项] [动作] 文件名` 
>
> 选项： -n 一般sed命令会把所有数据输出到屏幕，如果加入此选择，则会把经过sed命令处理的行输出到屏幕。
>
> -e 允许对输入数据应用多条sed命令编辑
>
> -i 用sed的修改结果直接修改读取数据的文件，而不是由屏幕输出
>
> ![image-20211123212819182](linux%E5%9F%BA%E7%A1%80/image-20211123212819182.png)
>
> 
>
> 



#### 9.4.3 字符处理命令

> sort 排序命令
>
> `sort 【选项】 文件名` 
>
> 选项： 
>
> -f 忽略大小写
>
> -n 以数值型进行排序，默认使用字符串型排序
>
> -r 反向排序
>
> -t 指定分隔符，默认分隔符是制表符
>
> -k n[,m]：按照指定的字段范围排序。从第n字段开始，m字段结束（默认到行尾）

#### 9.4.4 条件判断

![image-20211124102610006](linux%E5%9F%BA%E7%A1%80/image-20211124102610006.png)

:ice_cream: `[ -e /root/install.log ]` 判断文件 

:ice_cream: `test -e /root/install.log` 判断文件

![image-20211124104857073](linux%E5%9F%BA%E7%A1%80/image-20211124104857073.png)

判断文件权限：

:ice_cream: `[ -w /root/student.txt ]`

>两个文件之间进行比较
>
>![image-20211124111816645](linux%E5%9F%BA%E7%A1%80/image-20211124111816645.png)

> 整数比较
>
> ![image-20211124112314879](linux%E5%9F%BA%E7%A1%80/image-20211124112314879.png)

> 字符串判断
>
> ![image-20211124112353364](linux%E5%9F%BA%E7%A1%80/image-20211124112353364.png)

> 多重条件判断
>
> ![image-20211124113103024](linux%E5%9F%BA%E7%A1%80/image-20211124113103024.png)





#### 9.4.5 流程控制

> if 语句
>
> ![image-20211124200045771](linux%E5%9F%BA%E7%A1%80/image-20211124200045771.png)
>
> ```shell
> if [条件判断式];then
> 程序
> fi
> 或者
> if [条件判断式]
>   then
>   程序
> fi
> 
> ```
>
> ```shell
> if [条件判断式]
>   then
>     条件成立时，执行的程序
>   else 
>     条件不成立时，执行的另外一个程序
> fi
> ```
>
> ```
> if [条件判断式1]
>   then
>     当条件判断式1成立时，执行程序1
> elif [条件判断式2]
>   then
>     当条件判断式2成立时，执行程序2
> else
> 	当所有条件都不成立时，最后执行此程序
> fi
> ```
>
> 

#### 9.4.6 case语句

```shell
case $变量名 in
"值1")
	如果变量的值等于值1，则执行程序1
	;;
"值2")
	如果变量的值等于值1，则执行程序1
	;;
*)
    如果变量的值都不是以上的值，则执行此程序。
    ;;
esac
```

#### 9.4.7 for循环

第一种语法：

```shell
for 变量in 值1 值2 值3...
  do
    程序
  done
```

第二种语法：

```shell
for((初始值;循环控制条件;变量变化))
  do
    程序
  done
```

#### 9.4.8 while 循环

```shell
while [条件判断式]
  do
    程序
  done
```

```shell
until [条件判断式] // 和while判断结果相反
  do
    程序
  done
```

## 10.服务分类

![image-20211127150325968](linux%E5%9F%BA%E7%A1%80/image-20211127150325968.png)

> 查看已安装的服务
>
> ```shell
> # RPM包安装的服务
> chkconfig --list
> 
> #源码包安装的服务
> 一般是在/usr/local/ 下
> ```
>
> 



