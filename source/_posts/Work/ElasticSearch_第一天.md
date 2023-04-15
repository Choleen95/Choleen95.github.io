---
title: Elasticsearch的学习之旅
categories: Elastic
abbrlink: 3aad248e
date: 2021-03-05 00:00:00
tags:
---
##### 版权申明：

本文仅适用于学习，更多内容请访问原创作者：		

<img src="http://img.yangjiapo.cn/%E6%9D%BE%E5%93%A5.jpg" alt="avatar" style="zoom:33%;" />	

微信公众号：江南一点雨		

博客：https://www.javaboy.org/

<!--more-->

## ElasticSearch简介

###	1.1 Lucene

Lucene是一个开源、免费、高性能、纯java编写的全文检索引擎，可以算作是开源邻域最好的全文检索工具包。适用于很多语言，如C++、C#、paython等。它是出名的Doug Cutting的作品，当然还有hadoop也是他的。



Lucene主要特点：

- 简单

- 跨语言

- 强大的搜素引擎

- 索引速度快

- 索引文件兼容不同平台



###	1.2 ElasticSearch

ElasticSearch是一个分布式、可扩展、近实时性的高性能搜索与数据分析引擎，是基于Java编写，通过进一步封装Lucene，开发者可以用RESTful API

操作全文搜索。	



整体上来说，ElasticSearch有三大功能：

- 数据搜集

- 数据分析

- 数据存储



ElasticSearch主要有如下特点：

1. 分布式实时文件存储

2. 实时分析的分布式搜索引擎

3. 高可扩展性

4. 可插拔的插件支持



所以可以有很多用例	

- 应用程序搜索

- 网站搜索



### 1.3ElasticSearch安装



首先是打开以下链接的Es官网，找到ElasticSearch：

[ES官网](https://www.elastic.co/cn/elasticsearch/)



ElasticSearch对应的JDK关系，如下链接：

[JDK对应关系](https://www.elastic.co/cn/support/matrix#matrix_jvm)





#### 1.3.1单节点安装



基本上正常的window、linux、mac等系统都可以使用。

<img src="http://img.yangjiapo.cn/es01.png" alt="avatar" style="zoom:50%;" />



在Centos7.x上解压文件后，解压后的目录含义如下：	



![avatar](http://img.yangjiapo.cn/es02.png)





| 目录    | 含义           |
| ------- | -------------- |
| modules | 依赖模块目录   |
| lib     | 第三方依赖库   |
| logs    | 输出日志目录   |
| plugins | 插件目录       |
| bin     | 可执行文件目录 |
| config  | 配置文件目录   |
| data    | 数据存储目录   |



启动方式：	

进入到bin目录下，直接执行:	



```bash
[#]./elasticsearch
```



- 启动过程中出现的问题	

![avatar](http://img.yangjiapo.cn/es03.png)	

​	

原因：为了安全不允许使用root用户启动，es5之后的都不能使用添加启动参数或者修改配置文件等方法启动了, 创建一个用户就可以了		

​	1. 创建用户choleen

```
[root#]adduser choleen
```

​	2. 创建用户密码，需要输入两次

```bash
[root#]passwd choleen
```

​	3. 将对应的文件夹权限赋给该用户

```
[root#]chown -R choleen elasticsearch-7.10.0
```

​	4. 切换至choleen用户

```
[root#]su choleen
```

​	5. 进入bin启动，看到started就算成功

```
[root#]./elasticsearch
```

​	6. [root#]curl ip:9200，若看到json数据说明启动成功

​		可能没有成功，去elasticsearch.yml文件，更改network.host: 0.0.0.0即可，冒号后面又空格。	

​	7. 在浏览器中输入localhost:9200访问，则需要更改配置文件。	

![avatar](http://img.yangjiapo.cn/es05.png)



当改完配置文件之后，会报如下错误：	

1. 最大文件描述4096对于es进程太小了，至少增加到65535

2. 最大线程数量3754,对于这个用户太低了，至少增加到46096

3. 最大虚拟内存65535太低，至少增加到262144

4. 这个默认的发现设置不适用与生产使用，至少seed_host、seed_prividers、initial_master_node其中必须配置一个



![avatar](http://img.yangjiapo.cn/es06.png)



解决方法：	

- 切换到root	

​	

```bash
#文件
[root#]echo "* soft nofile 65535" >> /etc/security/limits.conf	
[root#]echo "* hard nofile 65535" >> /etc/security/limits.conf
#内存锁，不限制
[root#]echo "* soft memlock unlimited" >> /etc/security/limits.conf
[root#]echo "* hard memlock unlimited" >> /etc/security/limits.conf
[root#]echo "vm.max_map_count=262144" >> /etc/sysctl.conf
# 进程
[root#]echo "* soft nproc 4096" >> /etc/security/limits.d/20-nproc.conf 
[root#] echo "* hard nproc 4096" >> /etc/security/limits.d/20-nproc.conf 
[root#] sysctl -p
```



- 对于jdk环境变量	

jdk环境变量可以加在环境中，加es包里自带的。这个环境变量若不加，7.x版本的es启动会尝试在自己的程序所在路径获取jdk环境。若

要在这台机器启动kibana，可以加上，或者直接加到kibana启动脚本中。		

```bash
[root#]cat >> /etc/profile.d/java.sh <<EOF
> export JAVA_ES_HOME="/home/software/elasticsearch-7.10.0/jdk"
> export PATH=\$JAVA_ES_HOME/bin:\$PATH
> export CLASSPATH=.:\$JAVA_ES_HOME/lib/dt.jar:\$JAVA_ES_HOME/lib/tools.jar
> EOF
```



- 对于config/elasticsearch.yml配置文件修改	

```bash
node.name: node-1
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
# head插件需要这两个配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
```

当看到started时，启动成功。

![avatar](http://img.yangjiapo.cn/es04.png)	

在内网执行,出现json字符串则成功

```bash
[root#]crul ip:9200
```

![avatar](http://img.yangjiapo.cn/es08.png)