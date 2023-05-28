---
title: nodejs启动服务失败
tags: nodeJs
categories: WEB
abbrlink: a395ebd
date: 2020-03-07 13:10:54
---
nodejs启动服务过程中难免遇到一些问题，此次我就遇到了node-sass安装后环境发生变化的问题。
<!--more-->
### 1.第一次提示 ###
This usually happens because your environment has changed since running `npm install`
百度说是node-sass安装后环境发生变化。需要重新编译或安装node-sass

### 解决 ###
- 1.在项目根路径打开管理员模式的**cmd**
- 2  `npm rebuild node-sass `
* 2.1 提示**Cannot download "https://github.com/sass/node-sass/releases/download/v4.13.1/win32-x64-72_binding.node":**
* 3.设置淘宝下载 `npm i node-sass --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/`
* ok,启动node服务成功