---
title: CentOs7安装Redis5.0.7
categories: 服务器
abbrlink: 3b63363d
date: 2020-09-05 00:00:00
tags:
---
### 安装Ruby语言

0.查看哪些是稳定版

http://www.ruby-lang.org/zh_cn/downloads/

1.网站下载

https://cache.ruby-china.com/pub/ruby/ruby-2.6.6.tar.xz

2.也可以wget下载

```bash
wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-6.6.6.tar.gz
```
<!--more-->
3.若没有xz，先安装xz

```bash
xz ruby-2.6.6.tar.xz
tar -vxf ruby-2.6.6.tar
mv ruby-2.6.6 /usr/local/
mkdir ruby
cd ruby-2.6.6/
.configure --prefix=/usr/local/ruby
make
make install
./ruby -v
```

4.上面没有问题，配置环境

```bash
echo "export PATH=$PATH:/usr/local/ruby/bin" >> /etc/profile
source /etc/profile
ruby -v
```

### 安装redis5.0.7

1.网站下载

http://download.redis.io/releases/

2.解压

```bash
tar -vxf redis-5.0.7.tar.gz
mv redis-5.0.7 /usr/local/
mkdir redis
```

3.简易配置

```bash
cd redis-5.0.7/
vim redis.conf
----ctrl+ins复制
#bind 127.0.0.1
 protected-mode no
 timeout 3600
 daemonize yes
```

4.安装

```bash
cd redis-5.0.7/
make
make install prefix=/usr/local/redis
cd ../redis
mkdir etc
cd etc
mkdir log
cp /usr/local/redis-5.0.7/redis.conf /usr/local/redis/etc/
```

5.启动

```bash
cd /usr/local/redis/bin
./redis-server ../etc/redis.conf
./redis-cli -c -p 6379
```

![avatar](http://img.yangjiapo.cn/723-2.png)

### 集群配置

1.配置redis.conf

```bash
cd /usr/local/redis
mkdir etc
mkdir log
cd etc
mkdir 8000 8001 8002 8003 8004 8005
cp /usr/local/redis-5.0.7/redis.conf /usr/local/redis/etc/8000/
vim 8000/redis.conf
--
logfile "/usr/local/redis/log/6379.log"
dbfilename dump8000.rdb
appendonly yes
appendfilename "appendonly6379.aof"
cluster-enabled yes
cluster-config-file nodes_8000.conf 
cluster-node-timeout 15000 
:%s/6379/8000/g
```

![avatar](http://img.yangjiapo.cn/723-1.png)

**2.创建集群，8003，8004，8005之后分**

![avatar](http://img.yangjiapo.cn/724-2.png)

```bash
redis-cli --cluster create 192.168.246.130:8000 192.168.246.130:8001
192.168.246.130:8002
```

**3.登录**

```bash
cluster info
cluster nodes
```

![avator](http://img.yangjiapo.cn/724-1.png)

**4.动态添加节点，启动8003，8004，8005**

```bash
redis-cli --cluster add-node 192.168.246.130:8005 192.168.246.130:8000
redis-cli -c -p 8000
cluster info 
cluster nodes
```

![avatar](http://img.yangjiapo.cn/724-3.png)

查看状态

![avatar](http://img.yangjiapo.cn/724-3.png)

**5.动态添加从节点**

```bash
redis-cli --cluster add-node 192.168.246.130:8003 192.168.246.130:8000 --cluster-slave --cluster-master-id 25f77d1d595a1b7bad8f14058914792fcc3bad4a
redis-cli --cluster add-node 192.168.246.130:8004 192.168.246.130:8001 --cluster-slave --cluster-master-id 5e2633717488152e8e8ad640b73b5b937e1d08aa
redis-cli --cluster add-node 192.168.246.130:8005 192.168.246.130:8002 --cluster-slave --cluster-master-id 75c81e6f02e8e9cd32235b65a35f6f54d08b01b1
```

**6.一键添加节点**

```bash
redis-cli --cluster add-node 192.168.246.130:8000 192.168.246.130:8001 192.168.246.130:8002 192.168.246.130:8003 192.168.246.130:8004 192.168.246.130:8005 --cluster-replicas 1
```

一主一丛

