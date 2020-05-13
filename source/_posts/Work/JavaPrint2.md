---
title: springboot的java打印票据-2
date: 2020-03-26 21:11
tags:
categories: SpringBoot
---
### **Java打印之httpClient服务**
前面我们配置了httpClient实例，现在我们配置服务，我们配置get和post请求，用于之后的请求。    
1.注入实例时，若required没有或是ture则都是默认有这个实例的，不然肯定失败!
<!--more-->
```javascript
@Service
public class HttpClientService {

    @Autowired(required=false)
    private CloseableHttpClient httpClient;

    @Autowired(required=false)
    private RequestConfig config;

}
```   
- 1.这里我们首先编写get请求不带参数，如果状态码200，则会返回body，若不是200，则返回null。     
```javascript
 public String doGet(String url) throws Exception {
        // 声明 http get 请求
        HttpGet httpGet = new HttpGet(url);

        // 装载配置信息
        httpGet.setConfig(config);

        // 发起请求
        CloseableHttpResponse response = this.httpClient.execute(httpGet);

        // 判断状态码是否为200
        if (response.getStatusLine().getStatusCode() == 200) {
            // 返回响应体的内容
            return EntityUtils.toString(response.getEntity(), "UTF-8");
        }
        return null;
    }
```     
- 2.然后编写get带参数的请求。这里的带参结构都是map结构，我们可以引入解析json字符串为实体的依赖。     
```javascript
public String doGet(String url, Map<String, Object> map) throws Exception {
        URIBuilder uriBuilder = new URIBuilder(url);

        if (map != null) {
            // 遍历map,拼接请求参数
            for (Map.Entry<String, Object> entry : map.entrySet()) {
                uriBuilder.setParameter(entry.getKey(), entry.getValue().toString());
            }
        }

        // 调用不带参数的get请求
        return this.doGet(uriBuilder.build().toString());

    }
```     
- 3.设置带参数的post请求        
```javascript
public HttpResult doPost(String url, Map<String, Object> map) throws Exception {
        // 声明httpPost请求
        HttpPost httpPost = new HttpPost(url);
        // 加入配置信息
        httpPost.setConfig(config);

        // 判断map是否为空，不为空则进行遍历，封装from表单对象
        if (map != null) {
            List<NameValuePair> list = new ArrayList<NameValuePair>();
            for (Map.Entry<String, Object> entry : map.entrySet()) {
                list.add(new BasicNameValuePair(entry.getKey(), entry.getValue().toString()));
            }
            // 构造from表单对象
            UrlEncodedFormEntity urlEncodedFormEntity = new UrlEncodedFormEntity(list, "UTF-8");

            // 把表单放到post里
            httpPost.setEntity(urlEncodedFormEntity);
        }

        // 发起请求
        CloseableHttpResponse response = this.httpClient.execute(httpPost);
        return new HttpResult(response.getStatusLine().getStatusCode(), EntityUtils.toString(
                response.getEntity(), "UTF-8"));
    }
```     
- 4.不带参数的请求     
```javascript
 public HttpResult doPost(String url) throws Exception {
        return this.doPost(url, null);
    }
```     
- 5.还可以是json格式的参数，进行post请求      
```javascript
public String doPostJson(String url, String json) throws Exception {
        // 创建http POST请求
        HttpPost httpPost = new HttpPost(url);
        httpPost.setConfig(config);
        
        if(null != json){
            //设置请求体为 字符串
            StringEntity stringEntity = new StringEntity(json,"UTF-8");
            httpPost.setEntity(stringEntity);
        }

        CloseableHttpResponse response = null;
        try {
            // 执行请求
            response = httpClient.execute(httpPost);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == 200) {
                return EntityUtils.toString(response.getEntity(), "UTF-8");
            }
        } finally {
            if (response != null) {
                response.close();
            }
        }
        return null;
}
```     
到此，我们就把httpClient设置好了。要在那里使用直接注入@Autowired。
### ***logger日志配置***
在项目中，我们会发生很多异常，和需要打印信息。这里就要配置日志了。在properties中我们配置一些基础信息：        
```javascript
# 日志文件
logging.file.path=D:/logs/PrintLog/  #文件路径 
logging.file.name=Print.log # 文件名
logging.file.max-size=100MB  #文件定义最大的大小
logging.file.max-history=7 # 日志保存的最长时间
logging.config=classpath:logging-config.xml  #配置文件路径
```     
- 1.配置logging-config.xml，里面有几个节点。logback配置之Configuration。中有几个属性。其中： 
    - 1.scan：属性为true，文件发生改变，将会重新加载，defaut=true
    - 2.scanPeriod --字面意思扫描时间段，设置监测配置文件是否有时间间隔，没有给出具体时间间隔，单位毫秒，默认为1分钟。      
    - 3.debug 属性为true时，将打印出logback的内部日志信息，默认为false。
- 2.Configuration的子节点：
    - 1.contextName 上下文名称。用于区分不同的程序记录，默认为defaut，这里没有配置，一旦配置，便不能修改
    - 2.变量property，有name和value属性，设置之后可以在上下文中${name}使用
这里只配置root，使用appender进行输出日志文件及内容
```javascript
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--设置存储路径变量-->
    <property name="LOG_HOME" value="D:/logs/PrintLog/"/>

    <!--控制台输出appender-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!--设置输出格式-->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <!--设置编码-->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--文件输出,时间窗口滚动-->
    <appender name="timeFileOutput" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--日志名,指定最新的文件名，其他文件名使用FileNamePattern -->
        <File>${LOG_HOME}/jiuluPrint.log</File>
        <!--文件滚动模式-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名,可设置文件类型为gz,开启文件压缩-->
            <FileNamePattern>${LOG_HOME}/info.%d{yyyy-MM-dd}.%i.log.gz</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>7</MaxHistory>
            <!--按大小分割同一天的-->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>

        <!--输出格式-->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <!--设置编码-->
            <charset>UTF-8</charset>
        </encoder>

    </appender>

    <!--指定基础的日志输出级别-->
    <root level="INFO">
        <!--appender将会添加到这个loger-->
        <appender-ref ref="console"/>
        <appender-ref ref="timeFileOutput"/>
    </root>
</configuration>
```     
这样启动之后就会在设置路径下生成日志文件了。启动之后就会更替默认的日志格式。
![avator](http://img.yangjiapo.cn/154225130220180327.jpg)       
[个人博客:山沉](www.yangjiapo.cn)     
[CSDN](https://mp.csdn.net/console/article)        
[博客园](https://www.cnblogs.com/Choleen/)