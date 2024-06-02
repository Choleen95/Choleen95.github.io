---
abbrlink: 151f3b96
title: 一、tienchin健身系统技术点复现–注解重复提交
categories: SpringBoot
date: 2024-06-01
tags: SpringBoot
---

对于开发系统中，我们很多时候，会有很多方法

<!-- more -->

- Token机制

  - 首先客户端请求服务端，获取一个 token，每一次请求都获取到一个全新的token，将token存入到redis中，然后将token返回给客户端。
  - 客户端将来携带刚刚返回的token去请求一个接口。
  - 服务端收到请求后，分为两种情况：
    - 如果 token 在 redis 中，直接删除该 token，然后继续处理业务请求。
    - 如果 token 不在 redis 中，说明 token 过期或者当前业务已经执行过了，那么此时就不执行业务逻辑。
  - 优势：实现简单
  - 劣势：多一个获取 token 的过程。

  

- 用 Redis 的 setnx

  - 客户端请求服务端，服务端将能代表请求唯一性的业务字段，通过 setnx 的方式存入 redis，并设置超时时间。
  - 判断 setnx 是否成功：
    - 成功：继续处理业务
    - 失败：表示业务已经执行过了

- 设置状态字段

  - 要处理的数据，有一个状态字段

- 锁机制

  - 乐观锁：数据库中增加版本号字段，每次更新都根据版本号来判断。更新之前先去查询要更新记录的版本号，第二步更新的时候，将版本号也作为查询条件

    - select versionCode from xxx where id = xxx;
    - update xxx set xxx = xxx where xxx = xxx and versionCode = xxx;

  - 悲观锁

    - 假设每一次拿数据都会被修改，所以直接上排他锁就行了

    - ```
      start;
      select * from xxx where xxx for update
      update xxx
      commit;
      [Gitee代码路径](https://gitee.com/Choleen95/tien-chin-demo.git)## 1、装饰者模式-HttpServletRequest获取Json字节流首先继承 `HttpServletRequestWrapper` 重写构造方法、getInputStream()、getReader()```javapublic class RepeatableReadRequestWrapper extends HttpServletRequestWrapper {    private final byte[] body;    public RepeatableReadRequestWrapper(HttpServletRequest request, HttpServletResponse response)            throws IOException {        super(request);        request.setCharacterEncoding("UTF-8");        response.setCharacterEncoding("UTF-8");        body = request.getReader().readLine().getBytes();    }    @Override    public BufferedReader getReader() throws IOException {        return new BufferedReader(new InputStreamReader(getInputStream()));    }    @Override    public ServletInputStream getInputStream() throws IOException {        ByteArrayInputStream inputStream = new ByteArrayInputStream(body);        return new ServletInputStream() {            @Override            public boolean isFinished() {                return false;            }            @Override            public boolean isReady() {                return false;            }            @Override            public void setReadListener(ReadListener readListener) {            }            @Override            public int read() throws IOException {                return inputStream.read();            }            @Override            public int available() throws IOException {                return body.length;            }        };    }
      ```

然后创建一个过滤器，若是 `application/json` 格式的则进入装饰者，否则用原来的。

## 2、创建过滤器

由于过滤器 > servlet，所以在这里我们设置一下，在后面的拦截器中获取字节流就没有问题。

```
public class RepeatSubmitFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
            FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        // 若是json进入装饰者的request
        if (StringUtils.startsWithIgnoreCase(request.getContentType(), "application/json")) {
            RepeatableReadRequestWrapper wrapper = new RepeatableReadRequestWrapper(
                    request, ((HttpServletResponse) servletResponse));
            filterChain.doFilter(wrapper, servletResponse);
            return;

        }
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

对于下面的 重服提交拦截器，我们使用抽象类的方式，可以重写判断方式

```
@Component
public abstract class RepeatSubmitInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws Exception {
        // 获取前端传得入参、拦截有 RepeatSubmit 注解得方法，判断是否重复提交
        if (handler instanceof HandlerMethod) {
            // HandlerMethod 就是我们写的请求接口
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            Method method = handlerMethod.getMethod();
            RepeatSubmit annotation = method.getAnnotation(RepeatSubmit.class);
            if (annotation != null) {
                // 对这个加注解得方法进行判断
                if (validateRepeatableSubmit(request, annotation)) {
                    // 若是重复提交
                    Map<String, Object> map = new HashMap<>();
                    map.put("status", 500);
                    map.put("message", annotation.message());
                    response.setContentType("application/json;charset=utf-8");
                    response.getWriter().write(new ObjectMapper().writeValueAsString(map));
                    return false;
                }
            }
        }
        return true;
    }

    /**
     * 抽象方法，后面继承此类，可以重写生成 key 的方式
     *
     * @param request 请求
     * @param repeatSubmit 注解
     * @author [山沉]
     * @date 2023/6/8
     * @return {@link boolean}
     */
    public abstract boolean validateRepeatableSubmit(HttpServletRequest request,
            RepeatSubmit repeatSubmit);
}
```

这里的 `HandlerMethod` 就是我们定义的 请求接口类 `controller`

这个类可以获得 注解，然后判断是否为空，然后去作重复提交判断。

## 3、定义判断重复提交方法

```
/**
 * 功能描述 基础可重复提交拦截器，是可以指定是存入 redis 中得 key 组成，比如 加入 url、header 等等
 *
 * @author [山沉]
 * @个人博客 [https://choleen95.github.io/]
 * @博客 [https://www.cnblogs.com/Choleen/]
 * @since [2023/6/7 7:45]
 */
@Component
public class RepeatableSubmitUrlInterceptorImpl extends RepeatSubmitInterceptor {

    private static final String REPEAT_PARAMS = "repeat_params";
    private static final String REPEAT_TIME = "repeat_time";
    private static final String REPEAT_SUBMIT_KEY = "repeat_submit_key";
    private static final String HEADER = "Authorization";

    @Autowired
    RedisCache redisCache;

    @Override
    public boolean validateRepeatableSubmit(HttpServletRequest request, RepeatSubmit repeatSubmit) {
        // 获取入参
        String newParams = "";
        // request 是 RepeatableReadRequestWrapper
        if (request instanceof RepeatableReadRequestWrapper) {
            try {
                newParams = request.getReader().readLine();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (StringUtils.isEmpty(newParams)) {
            try {
                newParams = new ObjectMapper().writeValueAsString(request.getParameterMap());
            } catch (JsonProcessingException e) {
                e.printStackTrace();
            }
        }
        // 构建新的newMap存入redis 且和redis中得redisMap比较
        Map<String, Object> newMap = new HashMap<>();
        newMap.put(REPEAT_PARAMS, newParams);
        newMap.put(REPEAT_TIME, System.currentTimeMillis());
        // 获取 repeat_sumbit_key
        String header = request.getHeader(HEADER);
        StringBuffer requestURL = request.getRequestURL();
        String cacheKey = REPEAT_SUBMIT_KEY + requestURL + header.replace("Beare", "");
        Object cacheValue = redisCache.getCacheValue(cacheKey);
        if (cacheValue != null) {
            // 转换成 Map
            Map<String, Object> oldMap = (Map<String, Object>) cacheValue;
            if (compareParams(newMap, oldMap) && compareTime(newMap, oldMap,
                    repeatSubmit.interval())) {
                return true;
            }

        }
        // 这里对没有存入redis 的请求，设置值
        redisCache.setCacheValue(cacheKey, newMap, repeatSubmit.interval(), TimeUnit.MILLISECONDS);
        return false;
    }

    private boolean compareTime(Map<String, Object> newMap, Map<String, Object> oldMap,
            int interval) {
        long newTime = (Long) newMap.get(REPEAT_TIME);
        long oldTime = (Long) oldMap.get(REPEAT_TIME);
        // 若时间间隔小于注解时间间隔 则属于重复提交
        return (newTime - oldTime) < interval;
    }

    private boolean compareParams(Map<String, Object> newMap, Map<String, Object> oldMap) {
        String newParams = (String) newMap.get(REPEAT_PARAMS);
        String oldParams = (String) oldMap.get(REPEAT_PARAMS);
        return newParams.equals(oldParams);
    }
}
```

这里判断 `request` 是否是自定义的 `HttpServeltRequestWrapper` 若是，可以获取 `BufferReader`，然后获取字符串。否则可以获取 key-value 形式的入参。

对于 `String header request.getHeader(HEADER)` 此行代码非常关键，因为是请求头中带着用户信息，这样每个请求进来都可以生成不一样的 key 存入redis，作有效的重复提交判断。

## 4、注入过滤器的Bean

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    RepeatSubmitInterceptor submitInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(submitInterceptor).addPathPatterns("/**");
    }

    @Bean
    FilterRegistrationBean<RepeatSubmitFilter> filterFilterRegistrationBean() {
        FilterRegistrationBean<RepeatSubmitFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new RepeatSubmitFilter());
        bean.addUrlPatterns("/*");
        return bean;
    }
}
```

若是 `FilterRegistrationBean` 不生成Bean，注入自定义的filer，不能生效，在实现 `WebMvcConfigurer` 接口，实现 `addInterCeptors` 注入拦截器，且拦截请求规则。

## 5、重复提交注解

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RepeatSubmit {

    /**
     * 两个时间得间隔，毫秒
     */
    int interval() default 5000;

    /**
     * 重复提交时得文本
     */
    String message() default "单位时间内，不可重复提交";
}
```

在 `RepeatSubmitInterceptor` 中对于是否属于重复提交，就是 redis 中存储的系统时间，和当前时间作对比，若小于 RepeatSubmit 注解中的 interval 时间间隔，获取入参相同，就是重复提交。