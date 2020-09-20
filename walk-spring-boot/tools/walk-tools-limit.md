# walk-tools-limit 服务限流组件使用说明

## 总体介绍

limit组件用于对某个服务进行限流，并可通过指定key来设置基于某key的限流方式，组件使用注解`@WRateLimiter`声明一个服务使用

## 引入

- 使用maven配置

  ````xml
  <dependency>
      <groupId>org.walkframework.boot</groupId>
      <artifactId>walk-tools-limit</artifactId>
      <version>1.0.1</version>
  </dependency>
  ````

-  依赖spring boot运行时环境及walk基础依赖和缓存依赖，因为默认的滑动窗口配置是以redis实现的

  ````xml
  <dependency>
      <groupId>org.walkframework.boot</groupId>
      <artifactId>walk-starter-base</artifactId>
      <version>1.7.9</version>
  </dependency>
  <dependency>
      <groupId>org.walkframework.boot</groupId>
      <artifactId>walk-cache-starter</artifactId>
      <version>1.7.6</version>
  </dependency>
  ````

  

## 限流配置

组件默认使用`@WRateLimiter`注解中声明的配置达到限流目的，也可实现`org.walkframework.boot.tools.limit.service.RateLimiterConfigLoader`进行自定义配置信息

- `@WRateLimiter`具体配置：

```java
/**
 * 配置项，如果配置了，就从配置里取，如果未配置，则根据类名和方法名自动拼装
 */
String configItem() default "";

/**
 * <p>limit key，默认使用全部入参，可使用spel表达式配置key抽取规则如'args[0]['someKey']'，可用上下文参见：org.walkframework.boot.tools.limit.bean.LimitContext</p>
 */
String key() default "";

/**
 * <p>自定义key generator获取limit key，实现org.walkframework.boot.tools.limit.service.LimiterKeyGenerator，此处配置该服务名</p>
 */
String keyGenerator() default "";

/**
 * <p>总流量，不按key统计</p>
 */
double totalRate() default 0D;

/**
 * <p>按key配置的流量</p>
 */
double keyRate() default 0D;

/**
 * <p>滑动窗口宽度，单位秒，默认1分钟</p>
 */
long slidingWindow() default 60;
```

- 自定义配置，通过实现`RateLimiterConfigLoader`加载配置，可覆盖注解中除configItem外的其他所有配置

- 通过实现`org.walkframework.boot.tools.limit.service.SpecifyKeyRateLoader`，可对key的某个值进行单独配置流量，如配置-1屏蔽访问，配置0放开限流等 (1.0.1之后增加)

## 限流后处理

- 默认抛出`org.walkframework.boot.tools.limit.exception.RateLimitedException`异常提示不允许频繁操作
- 可自定义服务实现`org.walkframework.boot.tools.limit.service.AfterLimited`接口自行处理，如上报限流信息进行统计、返回一个自定义结果等。



## 
| [<-分布式追踪](https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-tracer) | [回列表](https://gaiyinaizhi.github.io/walk-spring-boot/index) |
| ------------------- | ------------------------------------------------------------ |
