---
abbrlink: baffc801
title: Resilience4j结合微服务
categories: SpringBoot
date: 2024-06-01
tags: SpringBoot
---

### 1、retry未生效

由于支持aop，所以要引入aop的依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

<!-- more -->

### 2、circuitBreaker异常

- Error creating bean with name 'io.github.resilience4j.circuitbreaker.autoconfigure.CircuitBreakerConfigurationOnMissingBean':

是因为我用的版本1.3.1，但使用了

![image-20240528065236516](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240528065236516.png)

应该是版本问题，引入依赖resilience4j-spring-boot2版本为1.3.1，但里面为0.13.2。后面我重新引入了circuitbreaker 1.3.1 版本，才有了 *SlidingWindowType* 这个类。

![image-20240528065622580](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240528065622580.png)

原来是我在父pom中规定了版本为0.13.2，改过来了。

![image-20240528070021447](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240528070021447.png)

在resilience4j中，变成了1.3.1版本

![image-20240528070202379](C:\Users\Cho\AppData\Roaming\Typora\typora-user-images\image-20240528070202379.png)

### 3、提示找不到 fallbackMethod中的降级方法

@CircuitBreaker 注解中的 name  属性用来指定 circuitbreaker 配置， fallbackMethod 属性用来指定服务降级的方法，**服务降级方法中，需要添加异常参数 Throwable** 

circuitbreaker 和 retry 搭配使用



### 4、idea推送代码一直提示邮箱与提交者不一致

由于新人入场，临时用的其他人的账号在提交代码，所以一直在切换账号。

解决方法：

- 删除windows凭据
- git config --global user.name ''
- git config --global user.email ''
- 重启电脑

以上是我先做的准备，此时git已经替换为设置的账号。用 git 原生命令推送，可以推，但用 idea 就不行。

```bash
git push origin xxx.git
```

**解决方法：**

- git clone 一份代码，把 *.git* 替换掉本地工程下的 *.git*
- idea 在settings 中的 appearence 下的 System settings 下的 **passwords** 设置为不保存密码

这样就可以重新用idea推送远程分支