### Spring Boot

##### Spring maven包相关
[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web): SpringMVC相关依赖，Server容器采用Tomcat
[spring-boot-autoconfigure](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-autoconfigure)：SpringBoot自动化配置
[spring-boot-starter-data-redis](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis): Redis Lettuce 相关
[spring-boot-starter-data-jpa](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)： JPA相关 由于国内用的人不多并且用不好  因此国内很少使用JPA 一般使用[Mybatis](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter) 或者 [Mybatis-plus](https://mvnrepository.com/artifact/com.baomidou/mybatis-plus)
[ spring-boot-starter-validation](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation):提供验证等方法

自用root pom 文件可见 [链接]()

#### SpringBoot 注解相关
通过在类上添加 @Configuration 注解，声明这是一个 Spring 配置类。
通过在方法上添加 @Bean 注解，声明该方法创建一个 Spring Bean。

@ConditionalOn 相关注解是条件类注解 表示当某条件符合的时候才生效

AutoConfiguration通过SpringFactoriesLoader 读取spring.factories注册

@Profile注解 通过pom中的profile来判定是否注册当前bean 一般来说用不上

##### 自定义注解
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

#### 热部署
某些脚本语言可使用热部署，例如PHP，只需要把代码文件放到nginx中即可完成部署，Java相关的需要编译成class文件才能给到执行，因此热部署方式目前由idea提供插件实现，或者采用spring-boot-devtools来实现，但是spring-boot-devtools不是热部署 而是快速重启

#### Lombok
常用的注解如下
|注解|描述|
|:--:|:--:|
|||
|||
|||
|||
|||
|||
|||
|||
|||
|||
|||
|||
|||