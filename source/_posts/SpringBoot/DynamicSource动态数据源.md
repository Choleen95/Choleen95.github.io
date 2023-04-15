---
title: DynamicSource动态数据源
tags: SpringBoot
categories: SpringBoot
abbrlink: 64a068f6
date: 2021-03-05 00:00:00
---
# 一、了解动态数据源场景
这里介绍一下一个类<br>
`AbstractRoutingDataSource`

这个类是我们切换数据源的核心，我们进入此类了解一番。<br>
<!--more-->
此类中的方法`determineTargetDataSource()` ，它返回的是一个key，不同的key实现可动态路由的数据源，下面是此方法的源码:<br>
```java
protected DataSource determineTargetDataSource() {
		Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
        //这个获得key的方法，可以由自定义的数据源类，继承AbstractRoutingDataSource类，重写
		Object lookupKey = determineCurrentLookupKey();
        //从数据源的Map中根据单进程获取的key，取出DataSource
		DataSource dataSource = this.resolvedDataSources.get(lookupKey);
		if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
            //若map中没有对应的数据源，则取默认的数据源
			dataSource = this.resolvedDefaultDataSource;
		}
		if (dataSource == null) {
			throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
		}
		return dataSource;
	}
```

从上面的代码中，我们知道，
- 这个数据源是封装到Map结构中的
- 需要设置默认数据源

<hr>
这两方面，需要我们在本地，创建一个类，去封装数据源。

# 二、封装数据源到Map<Object,DataSource>和设置默认数据源
在Demo中创建一个类`DynamicDatasourceConfiguration`;<br>
对于此类，我们需要从application.yml中获取配置参数，即数据源配置，比如url、username、password、driver-class-name等。<br>
下面有两种方法获取配置参数：<br>
- 实现`EnvironmentAware`接口, 实现`setEnvironment(Environment env)`此方法，从env参数中可获取数据库配置参数。
- 可以创建实体类接收，从此类中获取。可以用`@ConfigurationProperties(prefix = "")`注解。

我的Demo，从`Environment` 参数中获取，若数据源多，Springboot启动时，从spring容器中根据beans的name获取到，对应的值，数据源，加载一次即可。<br>
## 1.封装数据源类
如下代码：<br>
```java
/**
 * @description 数据源配置
 * @author 山沉
 * @公众号 九月的山沉
 * @微信号 Applewith520
 * @个人博客 Choleen95.github.com
 * @博客 https://www.cnblogs.com/Choleen/
 * @since 2021/1/1 22:02
 */
@Configuration
@Component
@MapperScan(basePackages = {"com.example.es.mapper"}, sqlSessionFactoryRef = "sqlSessionFactory")
@PropertySource(value = "classpath:application.yml")
public class DynamicDatasourceConfiguration implements EnvironmentAware{
    private static final Logger logger = LoggerFactory.getLogger(DynamicDatasourceRegister.class);


    private static  String PREFIX = "spring.datasource";

    @Override
    public void setEnvironment(Environment env) {

        init(env);
    }
    //目标数据源
    private Map<Object, Object> targetDataSources = new HashMap<>();
    //默认数据源
    private Object defaultTargetDataSource;

    @Bean("dynamicDataSource")
    public DynamicDatasource init(Environment env){
        String datasourceNames = env.getProperty(PREFIX+".dynamicNames");
        MyAssert.isEmpty(datasourceNames,"dataSource is null");
        String[] array = datasourceNames.split(",");
        DynamicDatasource dynamicDatasource = new DynamicDatasource();
        if(array.length == 1){
            //只有一个
            String type = array[0];
            DataSource dataSource = buildDataSource(env, type);
            targetDataSources.put(type,dataSource);
            defaultTargetDataSource = dataSource;
        }else {
            //多个
            for (String s : array) {
                DataSource dataSource = buildDataSource(env, s);
                targetDataSources.put(s,dataSource);
            }
        }
       dynamicDatasource.setTargetDataSources(targetDataSources);
       if(defaultTargetDataSource == null){
           defaultTargetDataSource = targetDataSources.get(array[0]);
           DynamicDataSourceContextHolder.setRouteKey(array[0]);
       }
       dynamicDatasource.setDefaultTargetDataSource(defaultTargetDataSource);
        return dynamicDatasource;
    }


    public DataSource buildDataSource(Environment props, String type){
        String dbNames = PREFIX+"."+type+ ".";
        String url = props.getProperty(dbNames+"url");
        String username = props.getProperty(dbNames+"username");
        String password = props.getProperty(dbNames+"password");
        String driverClassName = props.getProperty(dbNames+"driver-class-name");
        logger.info("加载数据源--------------->{}",url);
        DruidDataSource dataSource = DruidDataSourceBuilder.create().build();
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driverClassName);
        return dataSource;
    }

    @Bean
    public PlatformTransactionManager txManager(DataSource dynamicDataSource) {
        return new DataSourceTransactionManager(dynamicDataSource);
    }

    @Bean("sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactoryBean(@Autowired @Qualifier("dynamicDataSource") DynamicDatasource dynamicDataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dynamicDataSource);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath*:mapper/**.xml"));//mapper文件
        return sqlSessionFactoryBean.getObject();
    }

}

```

## 2. 配置文件
在`application.yml`中：<br>
```yml
spring:
  datasource:
    dynamicNames: mv,test
    mv:
      url: jdbc:mysql:///movie?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver
    test:
      url: jdbc:mysql:///testdb?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver
    druid:
      db-type: com.alibaba.druid.pool.DruidDataSource
      initial-size: 10
      max-active: 100
      min-idle: 3
      max-wait: 5000
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 100
```

## 3、自定以参数、继承类的参数
### 3.1、这里先介绍一下`AbstractRoutingSource`中的参数：<br>
```java
public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean {

	@Nullable
	private Map<Object, Object> targetDataSources;//这个是目标数据源

	@Nullable
	private Object defaultTargetDataSource;//默认数据源

	private boolean lenientFallback = true;

	private DataSourceLookup dataSourceLookup = new JndiDataSourceLookup();

	@Nullable
	private Map<Object, DataSource> resolvedDataSources;//已经解析的目标数据源

	@Nullable
	private DataSource resolvedDefaultDataSource;//解析的默认数据源
```

在自定义类中参数，有默认数据源，和目标数据源：
```java
 private Map<Object, Object> targetDataSources = new HashMap<>();//目标数据源

private Object defaultTargetDataSource;//默认数据源
```

## 4、需要生成`SqlSessionFactory`的实例，交给Spring容器管理
数据源封装了，在`AbstracRoutingSource`中可以根据routeKey去解析过的`resolveDataSources`中根据key获取数据源，即到此可以获取数据源了。但是没有让进程之间规定谁去访问哪一个数据源，为了避免出现混乱，我们这里需要用到`ThreadLocal`。

# 三、用`ThreadLocal` 避免数据源访问混乱
自定义一个类，专门去获取数据源的key，即A进程的访问不影响B进程，避免B进程不知去找哪一个key，出现数据混乱。<br>

## 如下代码
```java
public class DynamicDataSourceRouteKey {

    private static ThreadLocal<String> routeKey = new ThreadLocal<>();
    //获取数据源名称
    public static String getRouteKey() {
        return routeKey.get();
    }
    //设置数据源名称
    public static void setRouteKey(String type) {
        routeKey.set(type);
    }
    //清除数据源名
    public static void clearContextHolder(){
        routeKey.remove();
    }
}
```

从外访问接口：
- 可以在拦截器中设置数据源名
- 若有子进程，要在子进程中设置，在执行查询，指定routeKey

到此使用`AbstractRoutingSource` 实现动态数据源的切换功能完成。<br>
代码地址master分支：<br>
[github-hillheavy地址](https://github.com/Choleen95/hillheavy)































