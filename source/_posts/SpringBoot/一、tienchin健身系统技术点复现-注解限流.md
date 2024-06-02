---
abbrlink: 8261d2d3
title: 一、tienchin健身系统技术点复现-注解限流
categories: SpringBoot
date: 2024-06-01
tags: SpringBoot
---


这个技术用到的点是 用Java代码执行 redis 的 lua 脚本，采用 请求接口方法 注解@RateLimiter ，前置通知拦截判断请求次数，做出限流操作。

[Gitee代码仓库-rate-limiter](https://gitee.com/Choleen95/tien-chin-demo.git)

<!-- more -->

### 1、application.yml 配置 redis参数

在 application.yml 中配置redis基本的参数

```
spring.redis.host=127.0.0.1
spring.redis.password=choleen
spring.redis.port=6379
```

### 2、配置 Redis 序列化key-value

对于 redis 存储或取出时，不配置用的是JDK的序列化，在redis中存储的key和value会有一些其他的东西在前面，比如



```
name:张三
jdk---> \dsff\dsfg\dfdfname:\dfds\dfds\ggds张三
```

这种可以用 RedisTemplate 去调用 get(key)，但是用命令行就不行了，在后面用 lua 表达式时，也相当于使用命令行，所以需要我们重新设置序列化

创建一个RedisConfig类

```
/**
 * 功能描述 配置redis 的序列化
 *  配置lua脚本加载
 *
 * @author [山沉]
 * @个人博客 [https://choleen95.github.io/]
 * @博客 [https://www.cnblogs.com/Choleen/]
 * @since [2023/6/5 23:11]
 */
@Configuration
public class RedisConfig {

    @Bean
    RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        // 定义序列化方式,默认JDK序列化，它会：/dd23/f45/443name:xxxx 这种形式
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        redisTemplate.setKeySerializer(serializer);
        redisTemplate.setValueSerializer(serializer);
        // hash类型的序列化也设置一下
        redisTemplate.setHashKeySerializer(serializer);
        redisTemplate.setHashValueSerializer(serializer);
        return redisTemplate;
    }

    @Bean
    DefaultRedisScript<Long> limitScript() {
        //加载lua脚本
        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setResultType(Long.class);
        script.setScriptSource(new ResourceScriptSource(new ClassPathResource("lua/RateLimiter.lua")));
        return script;
    }

}
```

### 3、创建@RateLimiter注解

```
/**
 * 限制接口访问次数的注解-搭配切面使用
 *
 * @author 山沉
 * @date 2023/6/5 23:00
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RateLimiter {
    /**
     * 限流key，即redis中存入的key前缀 rate_limiter:192.168.246.130:com.yangjiapo.rate_limiter.controller.HelloController.hello
     */
    String key() default "rate_limier:";
    /**
     * 限流次数
     */
    int count() default 100;
    /**
     * 限流时间，即60秒内大于100就限制
     */
    int time() default 60;
    /**
     * 限流类型
     */
    LimitType limitType() default LimitType.DEFAULT;

}
```

创建LimitType 限流类型

```
public enum LimitType {
    /**
     *根据请求者Ip进行限流
     */
    IP,
    /**
     * 默认策略全局限流
     */
    DEFAULT

}
```

### 4、定义切面RateLimiterAspectj

```
@Aspect
@Component
public class RateLimterAspectj {

    private static final Logger LOGGER = LoggerFactory.getLogger(RateLimterAspectj.class);
    @Autowired
    private RedisTemplate<Object, Object> redisTemplate;
    @Autowired
    private DefaultRedisScript<Long> redisScript;


    @Before("@annotation(rateLimiter)")
    public void before(JoinPoint jp, RateLimiter rateLimiter) throws RateLimitException {
        // 拼接 redis 的key
        String key = getCombineKey(jp, rateLimiter);
        int count = rateLimiter.count();
        int time = rateLimiter.time();
        // redis 执行lua脚本
        try {
            Long number = redisTemplate
                    .execute(redisScript, Collections.singletonList(key), time, count);
            if (number == null || number.intValue() > count) {
                // 超过限流阈值
                LOGGER.info("当前接口已达到最大限流次数");
                throw new RateLimitException("访问过于频繁，请稍后访问!");
            }
            LOGGER.info("一个时间窗内请求次数：{}, 当前请求次数：{}，缓存的key为：{}", count, number, key);
        } catch (Exception e) {
            throw e;

        }
    }


    /**
     * 这个方法就是获取 redis 存储的接口调用次数的key
     * rate_limiter:192.168.130.221-com.yangjiapo.rate_limiter.controller.HelloController.hello
     *
     * @param jp 参数
     * @param rateLimiter 注解
     * @author [山沉]
     * @date 2023/6/5
     * @return {@link String}
     */
    private String getCombineKey(JoinPoint jp, RateLimiter rateLimiter) {
        MethodSignature signature = (MethodSignature) jp.getSignature();
        StringBuffer buffer = new StringBuffer(rateLimiter.key());
        if (LimitType.IP == rateLimiter.limitType()) {
            HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder
                    .getRequestAttributes()).getRequest();
            // 获取IP
            buffer.append(IpUtils.getIpAddr(request)).append("-");
        }
        String methodName = signature.getMethod().getName();
        Method method = signature.getMethod();
        String classRouting = method.getDeclaringClass().getName();
        buffer.append(classRouting).append("-").append(methodName);
        return buffer.toString();
    }

}
```

这里用到的 RedisTemplate、DefaultRedisScript 都在 RedisConfig 配置类中初始化了。

由于现在 Idea 智能了许多，大家在创建 切面时，容易创建为 aspect 类，这种就要根据它的要求写切点、通知等，若已经创建了 aspect，但想用 class 方式，即把此类删掉重新创建即可。

### 5、创建RateLimiter.lua脚本

在 Resources 目录下，创建 lua 目录，然后创建 RateLimiter.lua 脚本。

```
local key = KEYS[1]
local time = tonumber(ARGV[1])
local count = tonumber(ARGV[2])
local current = redis.call('get', key)
if current and tonumber(current) > count then
    return tonumber(current);
end
current = redis.call('incr', key)
if tonumber(current) == 1 then
    redis.call('expire', key, time)
end
return tonumber(current)
```

这里是 定义了三个参数：redis的key，过期时间 time，单位时间内限流次数 count

流程：

- 当 current 存在，且大于 count 时，即返回当前从redis中获取的 value 值，即第几次请求。
- 若 current 小于 count 则 使用 redis 的自增 1
- 若 当前是第一次请求，则给一个redis的过期时间 time
- 最后返回当前请求次数 current