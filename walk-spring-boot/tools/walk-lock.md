# 分布式锁支持

## 说明

默认提供三种实现：无锁，数据库锁，redis锁，可自主选择，也可配置默认，可指定锁超时时间等
提供带返回参数调用及无返回调用

## 引入

引入最新版本（1.0.3/更新于2020/08/19)

```xml

<dependency>
    <groupId>org.walkframework.boot</groupId>
    <artifactId>walk-tools-lock</artifactId>
    <version>1.0.3</version>
</dependency>

```

## 使用

### 配置默认锁模式

```yaml
walk:
  lock:
    default-lock-type: none/redis/db # 选择默认使用哪种锁
    redis:
      timeout: 超时时间，单位毫秒
    db:
      profile: 使用哪个数据源作为数据库锁的数据源
      timeout: 超时时间，单位秒（待优化）
```

### 编码

使用`org.walkframework.boot.tools.lock.service.factory.LockFactory`相关api进行锁内编码

```java

boolean action是否执行 = LockFactory.doInLock(String lockKey, LockAction 无返回结果Action实现);
ResponseInfo 执行结果 = LockFactory.doInLock(String lockKey, LockActionWithResult<T> 带返回结果Action实现);
ResponseInfo 执行结果 = LockFactory.doInLock(String lockKey, LockActionWithResult<T> 带返回结果Action实现, String 指定锁类型, int 指定锁时间);
// 更多api见LockFactory api

```


---
<div style="display: flex;font-size: 14px">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-tools-limit"><--服务限流</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-sequence">分布式序列--></a>
  </div>
</div>