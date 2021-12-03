### Spring Boot

##### Spring maven包相关
[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web): SpringMVC相关依赖，Server容器采用Tomcat

[spring-boot-autoconfigure](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-autoconfigure)：SpringBoot自动化配置

[spring-boot-starter-data-redis](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis): Redis Lettuce 相关

[spring-boot-starter-data-jpa](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)： JPA相关 由于国内用的人不多并且用不好  因此国内很少使用JPA 一般使用[Mybatis](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter) 或者 [Mybatis-plus](https://mvnrepository.com/artifact/com.baomidou/mybatis-plus)

[ spring-boot-starter-validation](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation):提供验证等方法

自用root pom 文件可见 [链接]()

#### 1.1. SpringBoot 注解相关
通过在类上添加 @Configuration 注解，声明这是一个 Spring 配置类。
通过在方法上添加 @Bean 注解，声明该方法创建一个 Spring Bean。

@ConditionalOn 相关注解是条件类注解 表示当某条件符合的时候才生效

AutoConfiguration通过SpringFactoriesLoader 读取spring.factories注册

@Profile注解 通过pom中的profile来判定是否注册当前bean 一般来说用不上

##### 1.1.1. 自定义注解
```
/**
 * @description: 自定义注解, 用于控制层统一处理封装结果, 异常, 日志和告警功能;
 * 同时集成了@RestController注解功能,后续接口开发只需使用@LoggableRestController注解用于Controller上
 **/
//集成@RestController
@RestController
//适用于类
@Target({ElementType.TYPE})
//只在运行时保留
@Retention(RetentionPolicy.RUNTIME)
//被javadoc记录
@Documented
public @interface LoggableRestController {
    @AliasFor(
            annotation = RestController.class
    )
    String value() default "";
}
```
```
@Aspect
@Log4j2
@Component
@DependsOn("springBeanUtils")
public class LoggableRestControllerHelper {

    @Autowired(required = false)
    private HttpServletRequest request;

    /**
     * 通過注解定义切点
     */
    @Pointcut("@within(LoggableRestController)")
    public void pointCut() {
    }

    /**
     * <p>使用AOP统一封装controller返回结果</p>
     *
     * @param point the date to format, not null
     * @return the object
     * @throws Throwable 系统异常
     */
    @ResponseBody
    @Around("pointCut() && @within(LoggableRestController)")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        Object respondData = point.proceed();

        // If the API return a RespVO object,
        // Return it directly here to avoid encode the result.
        // Currently, this is a work around for /version API
        if (respondData instanceof RespVO) {
            String json = JSONObject.toJSONString(respondData, SerializerFeature.DisableCircularReferenceDetect);
            return json;
        }

        RespVO res = RespVO.success(respondData);

        return JSONObject.toJSONString(res);
    }
}
```
这里的自定义注解 target表示用在什么类型之上，type类型包含class interface 等，然后使用Spring AOP切片 通过注解切分 随后在切片位置获取信息之后包装返回

#### 1.2. 热部署
某些脚本语言可使用热部署，例如PHP，只需要把代码文件放到nginx中即可完成部署，Java相关的需要编译成class文件才能给到执行，因此热部署方式目前由idea提供插件实现，或者采用spring-boot-devtools来实现，但是spring-boot-devtools不是热部署 而是快速重启

#### 1.3. Lombok
常用的注解如下
|注解|描述|
|:--:|:--:|
|@Data|1.为所有属性，添加 @Getter、@ToString、@EqualsAndHashCode 注解的效果 2.为非 final 修饰的属性，添加 @Setter 注解的效果 3.为 final 修改的属性，添加 @RequiredArgsConstructor 注解的效果|
|@Getter|添加在类或者属性上，设定get方法|
|@Setter|添加在类或者属性上，设定set方法|
|@Builder|添加在类上，生成构造者模式builder类|
|@Accessors|chain = true 链式编程|
|@Synchronized|添加在方法上 增加同步锁|
|@AllArgsConstructor、@RequiredArgsConstructor、@NoArgsConstructor|添加在类上，生成constructor方法|
|Log相关：@Log4j，@Log4j2等|添加在类上，支持日志收集|
|@SneakyThrows|添加在方法上，设定try catch|
|@NonNull|添加在方法参数 类属性上，校验是否null|

#### 1.4. MapStruct
MapStruct可用BeaaUtils 等其他方式替代 此处仅作记录 按照项目以及个人习惯使用 对象转换或者复制方法
MapStruct 是用于生成类型安全的 Bean 映射类的 Java 注解处理器。

你所要做的就是定义一个映射器接口，声明任何需要映射的方法。在编译过程中，MapStruct 将生成该接口的实现。此实现使用纯 Java 的方法调用源对象和目标对象之间进行映射，并非 Java 反射机制。

与手工编写映射代码相比，MapStruct 通过生成冗长且容易出错的代码来节省时间。在配置方法的约定之后，MapStruct 使用了合理的默认值，但在配置或实现特殊行为时将不再适用。

与动态映射框架相比，MapStruct 具有以下优点：

使用纯 Java 方法代替 Java 反射机制快速执行。
编译时类型安全：只能映射彼此的对象和属性，不能映射一个 Order 实体到一个 Customer DTO 中等等。
如果无法映射实体或属性，则在编译时清除错误报告。

使用方法：创建两个实体类，然后对于这两个实体类间新建一个convert 接口 convert接口上添加@Mapper注解，此注解会自动生成相关实体类转换的implements方法
接口样例：
```
@Mapper // <1>
public interface UserConvert {

    UserConvert INSTANCE = Mappers.getMapper(UserConvert.class); // <2>

    UserBO convert(UserDO userDO);

}
```
@Mapping注解可用于convert相关方法上 用于当实体类key不相等时使用 ```@Mapping(source="source", target="target")```

#### 1.5. 拦截器

拦截器一般用作access_token验证、用户信息获取之类情景下，通过对相关请求进行拦截，获取request header中一些信息来反查当前信息所对应的用户是否有权限操作，同时一般将用户信息存放至request attributes或者ThreadLocal中用作后续业务中使用，
```
// HandlerInterceptor.java

public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

}
```
PreHandle中return true 表示当前拦截器允许相关请求通过，反之return false
afterCompletion 实现请求完成之后的处理逻辑 诸如 释放资源、记录请求时间等操作 现在很少使用

多个 HandlerInterceptor 们，可以组成一个 Chain 拦截器链。那么，整个执行的过程，就变成：

- 首先，按照 HandlerInterceptor 链的正序，执行 #preHandle(...) 方法。
- 然后，执行 handler 的逻辑处理。
- 之后，按照 HandlerInterceptor 链的倒序，执行 #postHandle(...) 方法。
- 最后，按照 HandlerInterceptor 链的倒序，执行 #afterCompletion(...) 方法。

当出现异常时，任意一个interceptor抛出的Exception不影响其他interceptor的执行

#### 1.6. [WebFlux](https://www.v2ex.com/t/507828)

![](https://static.iocoder.cn/8d45025963c78e6cb191676fd6e9ec44)

WebFlux 响应式编程 类似RxJava的通知订阅模式 

对于现有的Servlet模式改动在于Controller层以及Filter层 Filter层改动较大 Controller层改动较小

同样webflux也可对返回值进行包装 因此Controller层的返回值 可仍旧沿用Servlet下的返回相关业务实体对象

样例Controller:
```
@GetMapping("/list")
public Flux<UserVO> list() {
    // 查询列表
    List<UserVO> result = new ArrayList<>();
    result.add(new UserVO().setId(1).setUsername("yudaoyuanma"));
    result.add(new UserVO().setId(2).setUsername("woshiyutou"));
    result.add(new UserVO().setId(3).setUsername("chifanshuijiao"));
    // 返回列表
    return Flux.fromIterable(result);
}

/**
* 获得指定用户编号的用户
*
* @param id 用户编号
* @return 用户
*/
@GetMapping("/get")
public Mono<UserVO> get(@RequestParam("id") Integer id) {
    // 查询用户
    UserVO user = new UserVO().setId(id).setUsername("username:" + id);
    // 返回
    return Mono.just(user);
}
```
上述样例中 返回列表使用Flux.fromInerable包装 对象使用Mono包装

```
// GlobalResponseBodyHandler.java

public class GlobalResponseBodyHandler extends ResponseBodyResultHandler {

    private static Logger LOGGER = LoggerFactory.getLogger(GlobalResponseBodyHandler.class);

    private static MethodParameter METHOD_PARAMETER_MONO_COMMON_RESULT;

    private static final CommonResult COMMON_RESULT_SUCCESS = CommonResult.success(null);

    static {
        try {
            // <1> 获得 METHOD_PARAMETER_MONO_COMMON_RESULT 。其中 -1 表示 `#methodForParams()` 方法的返回值
            METHOD_PARAMETER_MONO_COMMON_RESULT = new MethodParameter(
                    GlobalResponseBodyHandler.class.getDeclaredMethod("methodForParams"), -1);
        } catch (NoSuchMethodException e) {
            LOGGER.error("[static][获取 METHOD_PARAMETER_MONO_COMMON_RESULT 时，找不都方法");
            throw new RuntimeException(e);
        }
    }

    public GlobalResponseBodyHandler(List<HttpMessageWriter<?>> writers, RequestedContentTypeResolver resolver) {
        super(writers, resolver);
    }

    public GlobalResponseBodyHandler(List<HttpMessageWriter<?>> writers, RequestedContentTypeResolver resolver, ReactiveAdapterRegistry registry) {
        super(writers, resolver, registry);
    }

    @Override
    @SuppressWarnings("unchecked")
    public Mono<Void> handleResult(ServerWebExchange exchange, HandlerResult result) {
        Object returnValue = result.getReturnValue();
        Object body;
        // <1.1> 处理返回结果为 Mono 的情况
        if (returnValue instanceof Mono) {
            body = ((Mono<Object>) result.getReturnValue())
                    .map((Function<Object, Object>) GlobalResponseBodyHandler::wrapCommonResult)
                    .defaultIfEmpty(COMMON_RESULT_SUCCESS);
        // <1.2> 处理返回结果为 Flux 的情况
        } else if (returnValue instanceof Flux) {
            body = ((Flux<Object>) result.getReturnValue())
                    .collectList()
                    .map((Function<Object, Object>) GlobalResponseBodyHandler::wrapCommonResult)
                    .defaultIfEmpty(COMMON_RESULT_SUCCESS);
        // <1.3> 处理结果为其它类型
        } else {
            body = wrapCommonResult(returnValue);
        }
        // <2>
        return writeBody(body, METHOD_PARAMETER_MONO_COMMON_RESULT, exchange);
    }

    private static Mono<CommonResult> methodForParams() {
        return null;
    }

    private static CommonResult<?> wrapCommonResult(Object body) {
        // 如果已经是 CommonResult 类型，则直接返回
        if (body instanceof CommonResult) {
            return (CommonResult<?>) body;
        }
        // 如果不是，则包装成 CommonResult 类型
        return CommonResult.success(body);
    }

}
```
**个人不建议在生产环境使用webflux 由于与Servlet 相差较大 请评估后使用**

##### 1.6.1. WebFlux Filter

SpringMVC中可实现HandlerInterceptor接口来拦截请求，在WebFlux中 可实现WebFilter来实现相同的逻辑
```
// DemoWebFilterWebFilter.java

/**
 * Contract for interception-style, chained processing of Web requests that may
 * be used to implement cross-cutting, application-agnostic requirements such
 * as security, timeouts, and others.
 *
 * @author Rossen Stoyanchev
 * @since 5.0
 */
public interface WebFilter {

	/**
	 * Process the Web request and (optionally) delegate to the next
	 * {@code WebFilter} through the given {@link WebFilterChain}.
	 * @param exchange the current server exchange
	 * @param chain provides a way to delegate to the next filter
	 * @return {@code Mono<Void>} to indicate when request processing is complete
	 */
	Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain);

}
```
样例WebFilter
```
@Component
@Order(1)
public class DemoWebFilter implements WebFilter {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public Mono<Void> filter(ServerWebExchange serverWebExchange, WebFilterChain webFilterChain) {
        // <1> 继续执行请求
        return webFilterChain.filter(serverWebExchange)
                .doOnSuccess(new Consumer<Void>() { // <2> 执行成功后回调

                    @Override
                    public void accept(Void aVoid) {
                        logger.info("[accept][执行成功]");
                    }

                });
    }

}
```
Mono存在doOnSuccess方法表示执行成功之后 Mono还有诸如doOnError等方法 具体请参照[Reactor Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)

个人建议不采用上述的样例Filter中filter写法，建议下面的写法
```
@Override
public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
	ServerHttpRequest request = exchange.getRequest();
	// 如果是 CORS 跨域请求
	if (CorsUtils.isCorsRequest(request)) {
	    // 获得该接口的 CORS 跨域请求的配置
	    CorsConfiguration corsConfiguration = this.configSource.getCorsConfiguration(exchange);
	    if (corsConfiguration != null) {
		    // 执行 CORS 跨域请求的处理，
		    boolean isValid = this.processor.process(corsConfiguration, exchange);
		    // !isValid 表示，如果跨域请求的校验不通过
		    // CorsUtils.isPreFlightRequest(request) 表示，是 OPTIONS “预检请求”
		    if (!isValid || CorsUtils.isPreFlightRequest(request)) {
		        // 直接返回空的 Mono ，不进行后续的处理
		        return Mono.empty();
	        }
	    }
	}
	// 继续过滤器的处理
	return chain.filter(exchange);
}
```

上述样例在相对于SpringMVC Filter return false 的时候返回一个空的Mono来终止后续Filter执行，在当前Filter 通过时返回chain.filter(exchange)来继续执行剩下的Filter

##### 1.6.2. R2DBC
R2DBC(响应式的关系数据库连接)，是一种将响应式 API 引入 SQL 数据库的尝试。

此处记录的是 [jasync-sql](https://github.com/jasync-sql/jasync-sql)
- 基于 Netty 实现
- 异步、高性能、可靠的 PostgreSQL、MySQL 的驱动
- 使用 Kotlin 语言编写

```
// DatabaseConfiguration.java

@Configuration
@EnableTransactionManagement // 开启事务的支持
public class DatabaseConfiguration {

    @Bean
    public ConnectionFactory connectionFactory(R2dbcProperties properties) throws URISyntaxException {
        // 从 R2dbcProperties 中，解析出 host、port、database
        URI uri = new URI(properties.getUrl());
        String host = uri.getHost();
        int port = uri.getPort();
        String database = uri.getPath().substring(1); // 去掉首位的 / 斜杠
        // 创建 jasync Configuration 配置配置对象
        com.github.jasync.sql.db.Configuration configuration = new com.github.jasync.sql.db.Configuration(
                properties.getUsername(), host, port, properties.getPassword(), database);
        // 创建 JasyncConnectionFactory 对象
        return  new JasyncConnectionFactory(new MySQLConnectionFactory(configuration));
    }

    @Bean
    public ReactiveTransactionManager transactionManager(R2dbcProperties properties) throws URISyntaxException {
        return new R2dbcTransactionManager(this.connectionFactory(properties));
    }

}
```

#### 1.7. 分布式Session

多机部署的时候会出现用户登录到一台服务器上 之后tomcat检测cookie值不存在，然后生成一个sessionId并回写给Client，随后Client在另一台机器上登录，被nginx路由至另一台Tomcat，然而cookie所带的sessionId无法在这台Tomcat上找到对应session，这里又被创建了一个session，两台Tomcat需要做Session一致性，有以下方案：

- Tomcat做集群 session复制
- Nginx session 黏连
- Session存放至redis、mysql等数据库中

