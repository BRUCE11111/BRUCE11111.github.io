---
title: linux安装软件
top: false
cover: false
toc: true
mathjax: true
date: 2021-11-07 17:23:32
password:
summary:
tags:
categories:
---

# Ubuntu

## 1.Anaconda

1. 下载软件包 （<font color = "red">这个包有问题</font>）

`curl -O https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-s390x.sh`

注意这个是2021版本的。



### 1.1 加速conda下载

* 参考 [(21条消息) 【陆续排坑】解决conda下载速度慢的问题：更换国内源_黄元帅的博客-CSDN博客_conda更改下载源](https://blog.csdn.net/Xiao_Spring/article/details/109130663)

这里有三组国内源可供选择：

```bash
清华镜像：

https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/

中科大镜像：

https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/

上海交大镜像：

https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/main/
https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/free/
https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/conda-forge/
```

以清华镜像为例：

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
# 设置搜索时显示通道地址 从channel中安装包时显示channel的url，这样就可以知道包的安装来源
conda config --set show_channel_urls yes

```



* 还原下载源，删除上面的配置

```bash
conda config --remove-key channels
也可以使用conda config --remove-key channels XXX移除指定的链接
```







# Centos

## 1. Cmake

[(20条消息) CentOS7安装cmake-3.18.0_绮梦寒宵的博客-CSDN博客](https://blog.csdn.net/weixin_44901564/article/details/108225845)

### 2. Redis

[CentOS 8安装Redis的两种方式-四个空格 (4spaces.org)](https://www.4spaces.org/centos-8-install-redis/)

## 3. rabbitmq

安装： [centos8安装 rabbitmq - Sureing - 博客园 (cnblogs.com)](https://www.cnblogs.com/pengpengboshi/p/13614486.html)

### 4. fastdfs

[CentOS 8 安装 FastDFS 6.06 -阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/745317)

### 5. jupyter notebook

[配置Jupyter notebook远程访问，及jupyter notebook 插件 - 魔比之城 (scielib.com)](https://www.scielib.com/remote-jupyter-notebook.html)

nohup jupyter notebook --allow-root &

代码提示：

[(21条消息) JupyterNotebook代码提示与自动补齐_小明同学YYDS的博客-CSDN博客](https://blog.csdn.net/maoyuanming0806/article/details/109744284)

### 6. docker- compose

`curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

`chmod +x /usr/local/bin/docker-compose`

`ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

`docker-compose --version`

## 2. tvm

安装tvm好多坑，直接参考这个博主的，版本都按照他的来。不然很多地方编译的时候版本问题很抓狂。

[(21条消息) TVM：深度学习框架编译器的安装踩坑集_Lytain2022的博客-CSDN博客_tvm编译器](https://blog.csdn.net/mmphhh/article/details/116484914)



### 2.1错误信息

1. libtinfo.so.5

```bash
# 先查看tinfo自己OS上的版本
find / -name libtinfo.so* 
# 如果不匹配，直接将现在的版本转换成tvm要求的版本呢
ln -s /usr/lib64/libtinfo.so.6 /usr/lib64/libtinfo.so.5
```

2. 编译时报错 ld: can't find libtinfo.so libedit.so等问题

   ```bash
   find / -name libtinfo.so
   # 建立软连接'
   ln -s /root/anaconda3/lib/libtinfo.so /usr/lib/libtinfo.so
   ```

3. libbacktrace/configure: Permission denied

   chmod +x /path/to/tvm/3rdparty/libbacktrace/configure



### 4. gcc & cmake

对于 cmake 直接yum安装： `yum install cmake`



cmake .. 时 出现了can't find 问题

* 问题：

  * [CentOS 8: No matching repo to modify: PowerTools · Issue #101 · lanlin/notes (github.com)](https://github.com/lanlin/notes/issues/101)

    dnf search glibc

    dnf provides */libc.a

    dnf config-manager --set-enabled powertools

    dnf install glibc-static

    dnf install libstdc++-static

  





