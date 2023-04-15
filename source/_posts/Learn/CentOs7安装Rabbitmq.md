---
title: CentOs7安装Rabbitmq
categories: 服务器
abbrlink: dfc98018
date: 2020-09-05 00:00:00
tags:
---
## 装erlang语言

在rabbitmq官网查看对应版本号https://www.rabbitmq.com/which-erlang.html

![20200721.png](https://cdn.nlark.com/yuque/0/2020/png/1801717/1595412014205-4f60abdd-0b57-4181-9a00-ff033c1bddca.png)
<!--more-->
#### 安装依赖

```bash
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget gtk2-devel binutils-devel
```

#### erlang官网下载

1.http://erlang.org/download/   选择你所需要的版本下载

2.也可以这样下载，但是比较慢

```bash
wget http://erlang.org/download/opt_src_22.0.tar.gz
```

#### 解压

```bash
tar -zxvf opt_src_22.0.tar.gz
```

#### 移到local目录

```bash
mv opt_src_22.0 /usr/local/
```

#### 进入目录，编译，配置安装路径

```bash
cd /usr/local/opt_22.0/
mkdir ../erlang
./configure --prefix=/usr/local/erlang
```

编译时出现这个可以先忽略

![722-3.png](https://cdn.nlark.com/yuque/0/2020/png/1801717/1595419851771-c66bc9cd-01ba-41e8-9d27-85d6b33bb15b.png)

#### 安装

```bash
maike install
```

#### 添加环境变量，可以echo输入，也可以在/etc/profile中直接输入

```bash
echo 'export PATH=$PATH:/usr/local/erlang/bin' >> /etc/profile
source /etc/profile
erl
halt().b
```

### 安装Rabbitmq

GitHub下载地址https://github.com/rabbitmq/rabbitmq-server/releases/tag/v3.7.15

![0722-2.png](https://cdn.nlark.com/yuque/0/2020/png/1801717/1595424093675-6aaea226-1ad7-47fe-ad72-b5394feee7e3.png?x-oss-process=image%2Fresize%2Cw_1500)

下载

```bash
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-generic-unix-3.7.15.tar.xz
```

这里由于是tar.xz格式，需要安装

```bash
yum install -y xz
```

解压

```bash
/bin/xz -d rabbitmq-server-generic-unix-3.7.15.tar.xz
```

第二次解压

```bash
tar -xvf rabbitmq-server-generic-unix-3.7.15.tar
```

移动到local

```bash
mv rabbitmq_server-3.7.15/ /usr/local/
mv /usr/local/rabbitmq_server-3.7.15 rabbitmq
```

配置环境变量

```bash
echo 'export PATH=$PATH:/usr/local/rabbitmq/sbin' >> /etc/profile
```

刷新环境变量

```bash
source /etc/profile
```

创建配置目录

```bash
mkdir /etc/rabbitmq
```

### 启动、停止、网页端

```bash
rabbitmq-server -detached //后台启动
rabbitmqctl stop
rabbitmqctl status
```

打开5672/15672端口，或者关闭防火墙

#### web插件

```bash
rabbitmq-plugins enable rabbitmq_management
```

访问

```bash
http://127.0.0.1:1562/
```

### 用户管理

查看所有用户

```bash
rabbitmqctl list_users
```

添加一个用户

```bash
rabbitmqctl add_user admin 123456
```

配置权限

```bash
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```

查看用户权限

```bash
rabbitmqctl list_user_permissions admin
```

设置tag

```bash
rabbitmqctl set_user_tags admin administrator
```

删除用户

```bash
rabbitmqctl delete_user guest
```

##### 

