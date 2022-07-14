# walk-scheduler使用说明

本组件依赖表中的cron表达式进行定时任务的定义，可进行动态刷新，变更表配置后直接更新正在运行的任务

## 引用

- 当前版本2.0 (2020/6/30更新)

```xml
		<dependency>
			<groupId>org.walkframework.boot</groupId>
			<artifactId>walk-scheduler</artifactId>
			<version>${walk_scheduler_version}</version>
		</dependency>
```

## 使用说明

### 注解声明

```java
    // 对应schedule_task表中配置的task_name，从表中取任务信息
	@WScheduler("SHOP_VISITOR_TASK_EXAMPLE")
	public void work() {
		System.out.println("####shop visitor running..####");
		executeSubTasks();
	}
```

### 方法入参

方法定义可选参数如下，任务会根据方法入参类型自动匹配

```java
public void work(ScheduleTask scheduleTask, String jobKey, ScheduleTaskCheckpoint checkpoint)
```

`jobKey`的组成： `taskName-beanName-methodName`，如：`WALK-TESTER-ONE-schedulerOne-test1`

如果有方法重载，则后面加数字以保证`jobKey`不重复，可根据`jobKey`来判断此任务到底在哪个task下处理

### 任务配置

#### 依赖表

```mysql

DROP TABLE IF EXISTS `schedule_task`;

CREATE TABLE `schedule_task` (
  `task_name` varchar(64) NOT NULL COMMENT '任务名称',
  `task_module` varchar(64) DEFAULT NULL COMMENT '任务模块',
  `task_desc` varchar(128) DEFAULT NULL COMMENT '任务描述',
  `task_status` char(1) DEFAULT NULL COMMENT '0: 初始状态\n            1: 正常状态\n            2: 任务暂停\n            3: 任务异常终止\n            4: 终止',
  `cron_expr` varchar(100) DEFAULT NULL COMMENT '周期表达式',
  `error_policy` varchar(20) DEFAULT NULL COMMENT 'STOP         : 失败直接终止任务\n            RETRY_WHEN=ERRORCODES;TIMES=3;RETRY_TO_STOP  : 失败重试，当错误码为配置中的错误信息时进行重试，重试次数3次，重试3次依然失败则停止任务，配置根据需求变化\n            FAILED_TIMES_TO_STOP：连续失败多少次后停止任务，依赖failed_counter\n            CONTINUE ： 等待下一次执行',
  `failed_counter` int(11) DEFAULT '0' COMMENT '连续失败次数',
  `task_config` text COMMENT '参数等信息，动态配置',
  `log_keep_days` int(4) DEFAULT '60' COMMENT '任务运行日志保留时长',
  `lock_type` varchar(20) DEFAULT NULL COMMENT '分布式锁类型，默认使用应用默认的默认锁，可单独配置锁类型满足灵活需求',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `create_staff` varchar(20) DEFAULT NULL COMMENT '创建员工',
  PRIMARY KEY (`task_name`),
  KEY `idx_schedule_task_module` (`task_module`),
  KEY `idx_schedule_task_status` (`task_status`)
) COMMENT='任务配置表' ;

/*Table structure for table `schedule_task_log` */

DROP TABLE IF EXISTS `schedule_task_log`;

CREATE TABLE `schedule_task_log` (
  `task_name` varchar(64) NOT NULL COMMENT '任务名称',
  `execution_id` varchar(64) NOT NULL COMMENT '任务运行编码',
  `status` char(1) DEFAULT NULL COMMENT '执行状态',
  `start_time` datetime DEFAULT NULL COMMENT '执行开始时间',
  `end_time` datetime DEFAULT NULL COMMENT '执行结束时间',
  `fail_desc` varchar(500) DEFAULT NULL COMMENT '异常描述',
  `remark` varchar(128) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`execution_id`),
  KEY `index_task_log` (`task_name`)
)  COMMENT='任务运行日志表' ;

CREATE TABLE `schedule_task_registry` (
  `ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `task_module` varchar(64) NOT NULL COMMENT '任务模块',
  `registry` varchar(100) DEFAULT NULL COMMENT '注册信息，一般为IP或ip:port，可以自定义为任意信息',
  `is_coordinator` bit(1) DEFAULT b'0' COMMENT '是否为协调器，每个module只有一个协调器',
  `host` varchar(100) DEFAULT NULL COMMENT '当前注册应用的主机信息',
  `status` tinyint(4) DEFAULT NULL COMMENT '状态，1：在线，0：离线',
  `update_time` timestamp NOT NULL COMMENT '最近更新时间',
  PRIMARY KEY (`ID`),
  UNIQUE KEY `schedule_task_registry_un` (`registry`),
  KEY `schedule_task_registry_task_module_IDX` (`task_module`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='定时任务调度注册中心';

CREATE TABLE `schedule_task_distribution` (
  `ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `registry` varchar(100) NOT NULL COMMENT '注册信息',
  `task_key` varchar(100) NOT NULL COMMENT '任务主键',
  `execution_key` varchar(100) DEFAULT NULL COMMENT '当前任务执行标识',
  `distribution` text NOT NULL COMMENT '调度信息',
  `update_time` timestamp NOT NULL COMMENT '最近更新时间',
  PRIMARY KEY (`ID`),
  UNIQUE KEY `schedule_task_distribution_un` (`registry`,`task_key`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='定时任务调度信息分配结果';

```

#### 项目配置

```properties
# scheduler configuration. 此处配置和schedule_task表配置中的task_module对应
walk.module = shop-visitor-task
```

### 协调器及调度器支持

- since 2022/7/14 walk-scheduler version: 2.1.0
- 使用如下依赖代替原walk-scheduler依赖，即可增加协调和调度能力
```xml
            <dependency>
                <groupId>org.walkframework.boot</groupId>
                <artifactId>walk-scheduler-dispatchable</artifactId>
                <version>1.0</version>
            </dependency>
```

- schedule_task配置中增加lock_mode=`scheduleCoordinate`可将调度方式修改为协调器执行定时任务，可避免大批量的任务对redis的瞬时压力

- schedule_task配置中增加task_config配置参考如下，将lock_mode设置为none，即可打开动态调度能力，使所有节点各自分配某些任务进行处理，需要在应用中自行实现任务分配方法

```json
{"distributor": {"service":"scheduleDistributeServiceSample"}}
```
```java
// 实现该服务，将服务名配置到如上service字段中
public interface ScheduleDistributeService {

    /**
     * 执行调度
     *
     * @param scheduleDistributor 调度信息
     * @param allRegistry 全部注册信息
     * @return 调度结果列表
     */
    List<ScheduleDistribution> doDistribute(ScheduleDistributor scheduleDistributor, List<ScheduleRegistry> allRegistry);
}
```
```java
    // 每个节点都可以接收自己本次任务中需要处理哪些数据
    @WScheduler("TEST_DISTRIBUTE")
    public void testDistribute(ScheduleDistribution distribution) {
        logger.info("Beta接收到任务：" + distribution.getResult());
    }
```
#### 其他配置

使用`walk.scheduler.xxx`进行覆盖配置信息

```java
	// 任务刷新周期
    private int refreshInterval = 30;

    // 初始化延迟
    private int initialDelay = 10;

    // 配置数据库
    private String dbProfile = "base";

    // 线程池大小
    private int poolSize = 2 * Runtime.getRuntime().availableProcessors();

    // 默认的加锁模式，在schedule_task表未配置时
    private String lockMode = "none";
```

#### 分布式支持

默认单机版不需要锁，集群环境下才需要锁的实现，目前锁的选择一般为redis/zookeeper/数据库行锁

##### 内置锁

- 依赖walk-tools-lock组件，默认提供none/redis/db 3种锁模式

- db: 数据库行锁，由于定时任务执行频率不会太高，集群中数量也不会太大，因此使用此锁不会带来太多的数据库性能消耗，故内置此锁
- redis: 基于redis setNX的锁，适用于大批量锁使用时的应用
- none: 无锁，单机部署的场景不需要锁场景或者必须每台主机都要用的任务

##### 自定义锁

实现`org.walkframework.boot.tools.lock.service.LockService`，注入到Spring上下文中，以`xxLockService`命名，配置前缀作为lock_mode，如`RedisLockService`自动注册的beanName为redisLockService，则配置lock_mode为redis即可

---
<div style="display: flex;font-size: 14px">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/walk-mq"><--消息队列</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-imapper">快速参数映射及服务编排imapper--></a>
  </div>
</div>
