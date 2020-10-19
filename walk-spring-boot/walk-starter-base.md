# walk-starter-base使用说明

## 概述

顾名思义，此组件为walk spring boot系列的基础组件，提供Spring上下文的基础支持以及Spring启动器、SpringEL表达式语法解析支持、各种业务常用工具类等功能。

## 常用工具简述

### 上下文类

#### SpringContextHolder

提供Spring上下文的静态获取支持，便于在工具类中使用或避免不常用的bean的频繁注入

#### SpringPropertyHolder

提供Spring配置文件中的配置获取静态支持，便于在工具类中使用或避免不常用的属性的频繁注入

提供配置项解密支持

#### SpringElSupport

提供SpringEL表达式支持，walk组件中的动态语法支持均使用SpringEL进行书写，可动态配置默认函数，进行表达式解析支持

#### SimpleSpringContainer

简易Spring容器支持，默认启动Walk体系的基础starter，可实现快速测试或在无Spring容器状态时快速启动进行组件或业务bean的使用和支持（如spark executor中的组件使用的支持）

#### ConfiguredQualifier注解

使用此注解替换`@Qualifier`，支持如下方式根据配置动态选择实例进行注入，同时支持`#{}`语法，
从1.8.7版本开始支持
```java
@Autowired 
@ConfiguredQualifier("${ds.profile:default}StaticDAO")
private StaticDAO staticDAO;
```

### 工具类

#### LocalLRUCache

本地LRU策略缓存，可用于某些非分布式缓存场景的支持，提供缓存大小配置，避免本地缓存膨胀

#### AssertUtil

断言工具类

#### CollectionUtil

集合工具类

- 提供集合的简易构造如`CollectionUtil#buildKeyValueMap(key1, value1, key2, value2)`
- 提供基础的集合检测类如`isEmpty(java.util.Collection<?>)`, `isEmpty(java.util.Map<?,?>)`
- 提供集合转字符串

#### DateUtil

时间工具类，提供各种时间相关的操作

#### EncryptUtil

提供AES加解密支持

#### HostUtil

提供主机信息加载支持

#### MapUtil

提供Map的选择表达式支持

#### ResponseInfo/RequestInfo

可提供分布式追踪链路的公共请求、响应对象


#### SequenceUtil

提供本地唯一序列获取支持

### 异常类

#### BaseException

基础异常

#### CodeException

使用异常代码如“CUST-00001”类型的异常表现支持

### 其他

注解支持

- `Alias`  用来提供方法或类的别名 
- `ConditionalOnResourceOrProperty`可用来检测资源文件或配置属性任意存在一种即可配置指定的Bean



---
<div style="display: flex;font-size: 14px">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/walk-ehdb">数据库访问--></a>
  </div>
</div>