## Mock测试桩支持

### 概述

- 可以将暂时不可用或者不具备测试能力的外部调用使用mock测试桩屏蔽掉开发测试环节甚至生产上的影响
- 将联调之前的等待成本降至最低

### 测试服务配置获取

- 默认取Spring上下文中对应的service名称+".方法名"

- 可通过扩展方法重命名某些统一入口的服务名称，便于对不同的业务流程进行不同的测试结果响应

  ```java
  /**
       * <p>提供根据参数修改当前serviceName的入口</p>
       */
      @Override
      public String paramBasedServiceNameLocator(String serviceName, Object[] arguments) {
          if ("abilityShareCallService.call".equals(serviceName)) { // 重置能力平台的服务签名
              return "ability." + String.valueOf(arguments[0]);
          } else if ("motivationHttpService.call".equals(serviceName)) { // 重置激励中心的服务签名
              return "motivation." + String.valueOf(arguments[0]);
          }
          return serviceName;
      }
  ```

- 版本号可通过服务方法中配置`@MockConfig`注解进行设置，此处也可以重置服务名

- 版本号也可以通过配置`mock.properties`进行设置，如：

  ```properties
  walk.mock.version.mockOneInterface.testOne=1.0
  ```

  

### 测试桩结果配置

| 字段名       | 字段描述                                         |
| ------------ | ------------------------------------------------ |
| module       | 归属模块                                         |
| service_name | 服务签名，xxService.xxMethod                     |
| version      | 服务指定的mock版本，每个人都可以自定义自己的版本 |
| result       | 服务的mock结果返回                               |
| update_staff | 操作员工信息                                     |

#### 结果配置说明

1. 可以直接配置为json或普通字符串，组件自动转换为服务需要的出参类型
2. 如果配置为imapper的xml格式，则自动执行xml转换，转换为`Map<String, Object>`之后再转换为出参类型，imapper的详细用法见walk-imapper用法文档

```xml
<imapper>
    <id default="123123123"></id>
    <url default="xxxxxxxxx"></url>
    <amount>param1.amount</amount> <!--传入要做测试模拟返回的方法的入参1中的amount属性 -->
    <requestId default="123123123"></requestId>
</imapper>
```



### edas服务的测试桩配置

由于edas服务是通过接口注册的远程服务调用，本地无实现类，所以可以配置接口的默认实现类和mock_result的方式完成edas的测试结果响应

```properties
# 配置Edas服务的接口代理
walk.mock.interface.queryOrderInfoWoTouchService=com.chinaunicom.cbss2.ordercenter.woTouch.query.service.QueryOrderInfoWoTouchService
# 配置Edas服务中某方法的版本号
walk.mock.version.queryOrderInfoWoTouchService.userOrderQuery=1.0
```


---
<div style="display: flex">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-imapper"><--快速参数映射及服务编排imapper</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-tracer">分布式追踪--></a>
  </div>
</div>