# 分布式序列支持

## 说明

- 使用多种方式实现分布式序列
- 单机版实现见`org.walkframework.boot.util.SequenceUtil`，可生成单机顺序序列或分布式离散序列，不适用于作为数据库主键，
- 因此增加分布式序列支持，大概思路为每次通过分布式锁向目标存储申请特定步长的序列用于本地分配，耗尽后再次申请

## 引入

引入最新版本，指定实现方式引入，多种实现方式均继承自`walk-tool-sequence`框架

```xml

<!-- redis作为中间存储的实现 -->
<dependency>
    <groupId>org.walkframework.boot</groupId>
    <artifactId>walk-tools-sequence-redis</artifactId>
    <version>1.0.2</version>
</dependency>

<!-- 数据库作为中间存储的实现 -->
<dependency>
    <groupId>org.walkframework.boot</groupId>
    <artifactId>walk-tools-sequence-db</artifactId>
    <version>1.0.0</version>
</dependency>

```

## 编码

### 定义序列及序列的使用方式

实现接口，starter会自动扫描定义的Sequence对象，检查对象是否注册，未注册时会自动注册

```java
@Service
public class CommonSequenceService implements org.walkframework.boot.tools.sequence.SequenceDefinition {
	Sequence ORDER_ID_SEQ = new Sequence(0, 999999, 3000);
    
        @Autowired
    private SequenceService sequenceService;

    /** 获取16位订单ID */
    public String getOrderId16() {
        return sequenceService.getSequenceWithDatePrefix(ORDER_ID_SEQ.getName(), 6, DATE_FORMAT_LENGTH10);
    }
}
```


---
<div style="display: flex;font-size: 14px">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-lock"><--分布式锁</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
</div>