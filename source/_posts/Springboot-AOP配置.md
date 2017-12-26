---
title: Springboot AOP配置

date: 2017-12-26 13:25:35

categories:
- technology

tags:
- java
---
Springboot 整合aop做日志拦截，配置很简单。

1. 在pom.xml文件中引入aop相关的包
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
2. aop类配置示例
<!-- more -->
```java
@Aspect
@Component
public class LogAspect {

    private Logger logger = LoggerFactory.getLogger(LogAspect.class);

    @Pointcut("execution(* com.ht.yry.web.controller..*(..))")
    public void executeService(){}

   @Before("executeService() && @annotation(option)")
    public void beforePoint(JoinPoint joinPoint, ApiOperation option) {
       ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
       HttpServletRequest request = attributes.getRequest();
       //URL
       logger.info("url={}", request.getRequestURI());
       //method
       logger.info("method={}", request.getMethod());
       //ip
       logger.info("ip={}",request.getRemoteAddr());
       //类方法
       logger.info("class={} and method name = {}",joinPoint.getSignature().getDeclaringTypeName(),joinPoint.getSignature().getName());
       //参数
       logger.info("参数={}",joinPoint.getArgs(), joinPoint.getArgs());

       logger.info("apiOption={}", option.value());

	   // 获取方法所在类上的Api注解，例 @Api(tags="文件管理")
       Api annotation = AnnotationUtils
               .findAnnotation(joinPoint.getSignature().getDeclaringType(), Api.class);
       logger.info("api={}", String.join(",",annotation.tags()));
   }
}
```
