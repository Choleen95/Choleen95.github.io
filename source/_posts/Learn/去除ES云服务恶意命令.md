---
title: 去除ES云服务器恶意命令
categories: 服务器
abbrlink: '76531017'
date: 2020-09-05 00:00:00
tags:
---
## 云服务出现占用CPU-90%的恶意命令

#### 云服务提供的信息

![avator](http://img.yangjiapo.cn/phpupdate.png)
<!--more-->
先查看进程，选择top命令，查看结果：

![avator](http://img.yangjiapo.cn/806_3.png)

CPU占用率达到100%，且不是我执行的命令，果断找出执行路径。使用如下命令，通过 PID 获取对应文件的路径。然后，找到并删除对应的文件。

```bash
ls -l /proc/$pid/exe
```

这里的pid，我这里是14560。找到路径，删除命令或者文件。若存在恶意 minerd、tplink 进程，也查找出来删除。

![avator](http://img.yangjiapo.cn/806_1.png)

下面用的命令是进行删除出现这个提示的命令或文件：

```basic
cannot remove ‘phpupdate’: Operation not permitted
```

可以使用以下命令去删除文件

```
lsattr 文件/命令  --查看隐藏属性
chattr 隐藏属性 文件/命令
rm -rf 文件/命令  -- 重新执行删除
```

![avator](http://img.yangjiapo.cn/806_2.png)

最后在重新查看，进程，是否有类似很高的进程，又不是你自己的。