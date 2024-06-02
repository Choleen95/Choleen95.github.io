---
abbrlink: 3873cf6c
title: 2024还不会安装NodeJs
categories: SpringBoot
date: 2024-06-01 00:00:00
tags: SpringBoot
---

### 1、官网下载

由于本地是windows，所以选择 *Prebuilt Installer* 64位

[NodeJs官网下载 LTS长期支持版本](https://nodejs.org/en/download/prebuilt-installer)  

![image-20240602110924306](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240602110924306.png)

<!-- more -->

当然这里我用的 *Snipastate* 截屏软件，这里也配下载链接

[Snipasate下载链接](https://www.snipaste.com/download.html)

### 2、配置环境变量

由于是自己的电脑，可以直接设置系统环境变量，如果是其他地方，建议用用用户变量。

#### 2.1 全局配置

![image-20240602111319033](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240602111319033.png)

如果不配置全局，默认下载到 **C盘** 这就不太好了。

```bash
npm config set prefix  "你的路径node_global"
npm config set cache "你的路径node_cache"
```

系统变量: NODE_HOME

![image-20240602111912539](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240602111912539.png)

在 **Path** 中添加 node_modules

![image-20240602112019822](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240602112019822.png)

测试一下

```ba
npm install -g express
```

![image-20240602112120426](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240602112120426.png)

### 3、配置镜像

```bash
# 清空缓存
npm cache clean --force 
# 设置淘宝镜像
npm config set registry https:registry.npmmirror.com
# 检查是否设置成功
npm config get registry
```

![image-20240602112331059](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240602112331059.png)

### 4、安装 Hexo

```ba
npm install hexo-cli -g
```



![image-20240602112948684](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240602112948684.png)



### 5、NodeJS异常

```b
npm install -g cnpm -registry=https://registry.npm.taobao.org
```



npm error request to https://registry.npm.taobao.org/cnpm failed, reason: certificate has expired

淘宝NPM镜像站早就切换新域名了，老 **npm.taobao.org 和 registry.npm.taobao.org** 域名将于 2022年5月31日零时停止服务

使用正确的镜像源

```bash
npm config set registry https://registry.npmmirror.com
```



### 6、Webstorm不能使用 npm

在 settings > Languages & Frameworks > Node.js 目录下 新增node.exe

重启 WebStorm即可
