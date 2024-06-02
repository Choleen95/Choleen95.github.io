---
abbrlink: 5c28d527
title: Linux上nacos启动成功，但网页无法加载
categories: SpringBoot
date: 2024-06-01
tags: SpringBoot
---

### 1、docker查询

```
docker ps
```

<!-- more -->

### 2、查询firewall防火墙是否打开8848端口

```
firewall-cmd --query-port=8848/tcp
```

若是返回 **yes** 则是打开，若是返回 **FirewallD is not running** 则先打开防火墙。

```
systemctl start firewalld.service
```

### 3、打开8848端口

```
firewall-cmd --add-port=8848/tcp --permnent
```

### 4、重新载入端口

```
firewall-cmd --reload
```

### 5、最后查询端口是否打开及访问页面

```
firewall-cmd --query-port=8848/tcp
```

在网页端输入 **ip:8848/nacos**

