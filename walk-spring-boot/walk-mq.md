# walk-mq 消息组件使用说明

## 为什么使用

- 屏蔽各种底层mq的技术细节
- 统一mq使用api
- 方便进行技术选型的切换（`eg: 阿里云体系和其他体系对mq的切换`）

## 版本及引用

### 当前版本

1.10(2020/8/12)

### 引用方式

- 增加walk-mq依赖

```xml
<!-- 阿里ons消息队列组件支持 -->
<dependency>
    <groupId>org.walkframework.boot</groupId>
    <artifactId>walk-mq-ons-starter</artifactId>
    <version>${最新版本}</version>
</dependency>
<!-- kafka消息队列组件支持 -->
<dependency>
    <groupId>org.walkframework.boot</groupId>
    <artifactId>walk-mq-kafka-starter</artifactId>
    <version>${最新版本}</version>
</dependency>
```

## 快速开始

### 加入配置信息

指定消息类型，参考对应的Properties定义

``` properties
##ons配置，type声明对应到OnsProperties定义
walk.mq.base.type = ons  
walk.mq.base.servers = ip:port
walk.mq.base.access-key = xx
walk.mq.base.secret-key = xx
walk.mq.base.producer.producer-id = PID_XXXX
walk.mq.base.consumer.consumer-id = CID_XXXX
```

- 配置解释

  - `walk.mq`前缀为统一标识，用于区分组件配置的命名空间
  - `base`为profile设置，用于区别多份生产者或消费者，以上配置为Kafka配置，可以同时生成生产者和消费者实例，生产者命名为`baseMqOrderedProducer`

### 启用生产者

``` java
@Component
@EnableScheduling
public class Producer {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private MqOrderedProducer baseMqOrderedProducer;

    private int counter = 0;

    @Scheduled(fixedRate = 5000)
    public void produce() {
        String message = "this is a test message by order " + (++counter);
        logger.info("[test] start send message [{}]", message);
        baseMqOrderedProducer.send("test", "test", message);
    }
}
```

### 启用消费者监听

```java
@Component
@MqListener(topics = "${配置项}", cid="区分消费者ID或分组，可选，覆盖配置文件中的默认配置") // 声明要监听的topic列表
public class ConsumerListener extends AbstractOrderedMessageListener {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
    public void execute(String message) {
        logger.info("[test]start consume message [{}]", message);
    }
}
```

## 更多设置

- 开发支持

  ````properties
  # 抑制所有正常配置的消费者，只启用标记了force=true强制开启的消费者
  walk.mq.conf.disable-all = false
  ````

  ````java
  @MqListener（xxx, force=true）// 声明要调试的消费者为强制开启
  ````

- 所有配置项

```java
org.walkframework.boot.mq.kafka.config.KafkaProperties; // kafka配置项
org.walkframework.boot.mq.ons.config.OnsProperties; // ons配置项
```

---
<div style="display: flex;font-size: 14px">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/walk-cache"><--缓存</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/walk-scheduler">分布式定时任务--></a>
  </div>
</div>