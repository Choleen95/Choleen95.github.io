---
title: springboot的java打印票据
date: 2020-03-25 22:43
tags:
categories: SpringBoot
---
### **Java打印的几个步骤**
- 1构建springboot框架
- 2引入log日志
- 3构建java打印xml文件
- 4解析xml配置文件
- 5java打印
<!--more-->
由于需要，现在要实现一个java打印的程序。我们开始第一步构建springboot项目。    
#### **第一步**
打开idea，new 一个Project，选择Spring initializr 初始化一个springboot项目。在pom文件中按下alt + insert 选择Dependency使用idea的引入maven依赖。如下：   
```angular2html
        <!--基础依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!--httpClient连接-->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
        </dependency>
        <!--gogle的Zxing生成二维码-->
        <!-- https://mvnrepository.com/artifact/com.google.zxing/core -->
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>core</artifactId>
            <version>3.3.3</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.google.zxing/javase -->
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>javase</artifactId>
            <version>3.3.3</version>
        </dependency>
    <!--判断非空等的工具依赖-->
        <dependency>
            <groupId>me.javy</groupId>
            <artifactId>javy-helper</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--使用pojo解析xml文件所用-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.20</version>
        </dependency>
```     
下面是我的类结构图：      
![avator](http://img.yangjiapo.cn/jp1.png)
#### **配置httpClient**
1.httpClient连接可以远程访问服务器，获取数据，有时需要。所以这里也配一下。下面我们进行httpClient配置。在config包下建一个httpClient和httpCLientService，一个是构建对象一个是连接服务。    
在properties中写入httpclient的基础配置：  
```angular2html
#http配置服务
#最大连接数
http.maxTotal = 100
#并发数
http.defaultMaxPerRoute = 20
#创建连接的最长时间
http.connectTimeout=1000
#从连接池中获取到连接的最长时间
http.connectionRequestTimeout=500
#数据传输的最长时间
http.socketTimeout=10000
#提交请求前测试连接是否可用
http.staleConnectionCheckEnabled=true
```         
2.在httpClient往容器中注入Bean属性，采用注解@Value来实现如下：    
```angular2html
@Configuration
public class HttpClient {
	
    @Value("${http.maxTotal:1}")
    private Integer maxTotal;

    @Value("${http.defaultMaxPerRoute:1}")
    private Integer defaultMaxPerRoute;

    @Value("${http.connectTimeout:1}")
    private Integer connectTimeout;

    @Value("${http.connectionRequestTimeout:1}")
    private Integer connectionRequestTimeout;

    @Value("${http.socketTimeout:1}")
    private Integer socketTimeout;

    @Value("${http.staleConnectionCheckEnabled:true}")
    private boolean staleConnectionCheckEnabled;
}
```     
- 1.下面我们首先实列化一个连接池管理器，设置最大连接数，并发连接数。
```angular2html
     @Bean(name = "httpClientConnectionManager")
    public PoolingHttpClientConnectionManager getHttpClientConnectionManager(){
        PoolingHttpClientConnectionManager httpClientConnectionManager = new PoolingHttpClientConnectionManager();
        //最大连接数
        httpClientConnectionManager.setMaxTotal(maxTotal);
        //并发数
        httpClientConnectionManager.setDefaultMaxPerRoute(defaultMaxPerRoute);
        return httpClientConnectionManager;
    }
```     
- 2.实列化连接池，设置连接池管理器。我们需要用到spring的@Qualifier，其意为合格者，用于区分注解实例，参数名为我们的连接池管理器注解名称。      
```angular2html
 @Bean(name = "httpClientBuilder")
    public HttpClientBuilder getHttpClientBuilder(@Qualifier("httpClientConnectionManager")PoolingHttpClientConnectionManager httpClientConnectionManager){

        //HttpClientBuilder中的构造方法被protected修饰，所以这里不能直接使用new来实例化一个HttpClientBuilder，可以使用HttpClientBuilder提供的静态方法create()来获取HttpClientBuilder对象
        HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();

        httpClientBuilder.setConnectionManager(httpClientConnectionManager);

        return httpClientBuilder;
    }
```     
![avator](http://img.yangjiapo.cn/jp2.png)
- 3.注入连接池，获取client对象
```angular2html
    @Bean
    public CloseableHttpClient getCloseableHttpClient(@Qualifier("httpClientBuilder") HttpClientBuilder httpClientBuilder){
        return httpClientBuilder.build();
    }
```         
- 4.Builder是requestConfig中的一个内部类，通过requestConfig的custom方法获取builder的对象，设置builder的连接信息。   
```angular2html
    @Bean(name = "builder")
    public RequestConfig.Builder getBuilder(){
        RequestConfig.Builder builder = RequestConfig.custom();
        return builder.setConnectTimeout(connectTimeout)//创建连接的最长时间
                .setConnectionRequestTimeout(connectionRequestTimeout)//从连接池中获取的连接最长时间
                .setSocketTimeout(socketTimeout)//数据传输的最长时间
                .setStaleConnectionCheckEnabled(staleConnectionCheckEnabled);//提交请求前测试连接是否可用
    }
```     
- 5.用builder构建一个requestConfig对象
```angular2html
@Bean
    public RequestConfig getRequestConfig(@Qualifier("builder") RequestConfig.Builder builder){
        return builder.build();
    }
```         
到此httpclient基础信息初始化配置完成。