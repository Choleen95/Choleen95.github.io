---
title: hexo+icarus搭建的第一个博客
categories: 人生
abbrlink: 1ce21b5d
date: 2020-03-07 13:10:54
tags:
---
博客的搭建过程中，才发现自己会的东西挺少。今后会继续学习，持之以恒，参考了的一些资料，总算搭建了一个hexo+icarus实现的博客。
<!--more-->
### 所需内容 ###
* github账户一个
* nodejs安装(之后在安装hexo等)
* git for windows
#### 0.申请一个github账户
注册账户，完成之后生成一个GitHub的 ID.github.io的仓库。这个在之后上传静态资源，可以作为默认网址访问博客。
#### 1.nodejs安装 
这里不过多的叙述，或者在网上找资料，普通的安装。
### 2.hexo+icarus搭建博客
&emsp;这里推荐[小明同学的博客](https://www.cnblogs.com/liuxianan/p/build-blog-website-by-hexo-github.html)，丰富的记录了hexo搭建博客的所需事项。
当时域名的绑定与配置没有搞清楚，后来配置域名时，才知道在国内注册的域名要备案，后来又去备案，结果没有备案，照下面这样做，也能访问。(最好备案，各个网站的app备案更快.)
### 3.配置华为云域名 ###
* ![avatar](http://img.yangjiapo.cn/1.jpg)

* ![avatar](http://img.yangjiapo.cn/2.jpg)

* ![avatar](http://img.yangjiapo.cn/3.jpg)

* ![avatar](http://img.yangjiapo.cn/4.jpg)
在项目的source目录下建一个没有后缀的文件-CNAME，里面写入www.xxx.xx你的域名。
```
hexo clean
hexo generate
hexo deploy
```
只要过2到3分中即可用域名访问了。OK！