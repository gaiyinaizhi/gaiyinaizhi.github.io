{% raw %}
## ehdb使用说明

## 概述

针对spring jdbc template做使用增强，定义StaticDAO的API进行封装，形成简单易用的面向对象的操作和动态SQL的支持

## 引入

- 查看最新版本号

````xml
        <dependency>
            <groupId>org.walkframework.boot</groupId>
            <artifactId>walk-ehdb</artifactId>
            <version>3.6.0</version> <!-- 2020/09/01 -->
        </dependency>
````

## 生成语法文件

指定数据库连接，生成数据库中表对应的Bean和Repository(<font color=gray>Repository和Service可以不生成</font>)等文件

可在web工程增加test用例生成，使用已配置的数据库

```java
public class CreateBeans extends TestBase {

    @Autowired
    private DataSource baseDataSource;
    //private DataSource ordProdDataSource;

    @Test
    public void createBeans() throws Exception {
        // 1. 指定生成到哪个工程
        String basePath = "E:\\app-name\\src\\main\\java";  // 生成文件的路径
        // 2. 配置生成需要的2个字段
        String beanPackage = "org.walkframework.boot.scheduler.bean"; // bean生成的目录
        //String repositoryPackage = "com.asiainfo.ica.collect.service.atom.sys"; // repository生成的目录
        // 3. 设置截取表名前缀
        CreateBeanUtils.setSubTableName(true);
        // 4. 设置生成人员信息
        CreateBeanUtils.setUserName("mengqk");
        // 5. 设置数据源及路径配置
        CreateBeanUtils.setup(baseDataSource, basePath, beanPackage, null);
        // 生成表结构语法文件
        CreateBeanUtils.createBeans(false,
                "SCHEDULE_TASK_LOG",
                "SCHEDULE_TASK"
                );
    }
}

```

- 生成Bean对象示例：

````java
package org.walkframework.boot.scheduler.bean;

import java.util.Date;

import org.apache.commons.lang3.StringUtils;
import org.walkframework.boot.ehdb.bean.BaseBean;
import org.walkframework.boot.ehdb.annotation.*;

/**
 * 基于表SCHEDULE_TASK自动生成的JavaBean
 *
 * @since  [2020/06/29]
 * @author mengqk
 */
@Table("SCHEDULE_TASK")
public class ScheduleTask extends BaseBean implements java.io.Serializable {

	public ScheduleTask(){
		super();
		addPrimaryKey(Columns.TASK_NAME);
	}

	/**
	 * 任务名称
	 */
	@Column(Columns.TASK_NAME)
	private String taskName;

	/**
	 * 任务模块
	 */
	@Column(Columns.TASK_MODULE)
	private String taskModule;

	public String getTaskName() {
		return this.taskName;
	}

	public void setTaskName(String taskName) {
		this.taskName = taskName;
		recordUpdated(Columns.TASK_NAME);
	}

	public ScheduleTask letTaskName(String taskName) {
		setTaskName(taskName);
		return this;
	}

	public String getTaskModule() {
		return this.taskModule;
	}

	public void setTaskModule(String taskModule) {
		this.taskModule = taskModule;
		recordUpdated(Columns.TASK_MODULE);
	}

	public ScheduleTask letTaskModule(String taskModule) {
		setTaskModule(taskModule);
		return this;
	}


	public ScheduleTaskQueryBuilder queryBuilder() {
		return new ScheduleTaskQueryBuilder();
	}	
    public ScheduleTask setQueryColumns(String... columnNames) {
		for (String columnName : columnNames) {
			addQueryKey(columnName);
		}
		return this;
	}

	public ScheduleTask lOrderBy() {
		orderBy();
		return this;
	}
	public ScheduleTask lOrderBy(boolean isAsc) {
		orderBy(isAsc);
		return this;
	}
	public ScheduleTask lOrderBy(String columnName) {
		orderBy(columnName);
		return this;
	}
	public ScheduleTask lOrderBy(String columnName, boolean isAsc) {
		orderBy(columnName, isAsc);
		return this;
	}
	public ScheduleTask letGetLockMode() {
		setGetLock(true);
		return this;
	}
	public ScheduleTask letReplaceMode() {
		setReplaceMode(true);
		return this;
	}

	public class ScheduleTaskQueryBuilder implements java.io.Serializable {
		public ScheduleTaskQueryBuilder taskName() {
			ScheduleTask.this.addQueryKey(Columns.TASK_NAME);
			return this;
		}

		public ScheduleTaskQueryBuilder taskModule() {
			ScheduleTask.this.addQueryKey(Columns.TASK_MODULE);
			return this;
		}

		public ScheduleTask finishBuilder() {
			return ScheduleTask.this;
		}

	}

	public interface Columns {

		String TASK_NAME = "TASK_NAME";

		String TASK_MODULE = "TASK_MODULE";
	}
}
````





## 启动配置

- 数据源配置

````properties
# 基础配置, base即数据源的标识，在代码中使用baseStaticDAO注入操作类
walk.datasource.base.url=jdbc:mysql://xxx:3306/dbname?autoReconnect=true&amp;useUnicode=true&amp;characterEncoding=UTF-8
walk.datasource.base.username=dbname
walk.datasource.base.password=加密的密码或对应walk-security-pass的配置项
walk.datasource.base.driverClassName=com.mysql.jdbc.Driver

#druid连接池及监控配置
walk.datasource.base.initialSize=5
walk.datasource.base.minIdle=5
walk.datasource.base.maxActive=20
walk.datasource.base.maxWait=60000
walk.datasource.base.timeBetweenEvictionRunsMillis=60000
walk.datasource.base.minEvictableIdleTimeMillis=300000
walk.datasource.base.validationQuery=SELECT 1 FROM DUAL
walk.datasource.base.testWhileIdle=true
walk.datasource.base.testOnBorrow=false
walk.datasource.base.testOnReturn=false
walk.datasource.base.poolPreparedStatements=true
walk.datasource.base.maxPoolPreparedStatementPerConnectionSize=20
walk.datasource.base.filters=stat,wall
walk.datasource.base.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

````

- ehdb通用配置

````properties
# sql配置表，默认为SQL_CONFIG
walk.ehdb.config-table=WALK_SQL_CONFIG
# sql缓存时间，默认2小时
walk.ehdb.cache-time=7200000
# 配置事务监听目录
walk.ehdb.txPointcut = execution(* package.app.server.service..*.*(..))
# 配置事务监听的方法如下，默认监听的方法见下一小节
walk.ehdb.txMethod.add* = PROPAGATION_REQUIRED,-Throwable


## 其他配置见org.walkframework.boot.ehdb.config.EhDbConfig
````

- 默认事务监听的方法

```java
    String TX_METHOD_PROPERTY_DEFAULT = "PROPAGATION_REQUIRED,-Throwable";
    Map<String, Object> TX_METHODS_DEFAULT = new HashMap<String, Object>(){{
        put("add*", TX_METHOD_PROPERTY_DEFAULT);
        put("insert*", TX_METHOD_PROPERTY_DEFAULT);
        put("create*", TX_METHOD_PROPERTY_DEFAULT);
        put("execute*", TX_METHOD_PROPERTY_DEFAULT);
        put("save*", TX_METHOD_PROPERTY_DEFAULT);
        put("commit*", TX_METHOD_PROPERTY_DEFAULT);
        put("submit*", TX_METHOD_PROPERTY_DEFAULT);
        put("update*", TX_METHOD_PROPERTY_DEFAULT);
        put("delete*", TX_METHOD_PROPERTY_DEFAULT);
        put("do*", TX_METHOD_PROPERTY_DEFAULT);
    }};
```



## 1) 基于bean的基础操作

### 查询

````java
    @Autowired
    private StaticDAO baseStaticDAO;

    /** 基础查询用法，查询SCHEDULE_TASK表中TASK_MODULE=TEST并且TASK_NAME为TestTaskName的一条数据 */
    @Test
    public void baseQuery() {
        ScheduleTask scheduleTask = new ScheduleTask().letTaskName("TestTaskName").letTaskModule("TEST");
        ScheduleTask queryResult = baseStaticDAO.queryOne(scheduleTask);
        Assert.assertEquals(scheduleTask.getTaskName(), queryResult.getTaskName());
    }

    /** 分页查询，查询第一页数据，最多查询10行 */
    @Test
    public void pageQuery() {
        ScheduleTask scheduleTask = new ScheduleTask().letTaskName("TestTaskName").letTaskModule("TEST");
        PageResult<ScheduleTask> queryResult = baseStaticDAO.queryByPage(scheduleTask, new Pagination(1, 10));
    }
	/** 注意分页使用时，当查询第2页及以后时，构造Pagination时把total总条数带入，可以减少再次查询总数以提高性能 */
````

### 新增

```java
    /** 插入一条数据，内容为设置的3个字段 */
    @Test
    public void insert() {
        ScheduleTask scheduleTask = new ScheduleTask().letTaskName("TestTaskName")
            .letTaskModule("TEST").letCronExpr("* * 1 * * ?");
        baseStaticDAO.save(scheduleTask);
    }

    /** 批量插入10条数据，内容为设置的3个字段 */
    @Test
    public void batchInsert() {
        List<ScheduleTask> tasks = new ArrayList<>();
        ScheduleTask scheduleTask;
        for (int i = 0; i < 10; i++) {
            scheduleTask = new ScheduleTask()
                .letTaskName("TestTaskName" + i).letTaskModule("TEST")
                .letCronExpr("* * 1 * * ?");
            tasks.add(scheduleTask);
        }
        baseStaticDAO.saveBatch(tasks);
    }
```

### 更新

```java
    /** 按主键更新表 */
    @Test
    public void updateByPrimary() {
        ScheduleTask scheduleTask = new ScheduleTask()
            .letTaskName("TestTaskName")
            .letTaskModule("TEST").letCronExpr("* * 1 * * ?");
        baseStaticDAO.updateByPrimaryKeys(scheduleTask);
    }

    /** 按QueryBuilder设置的where字段taskName进行更新 */
    @Test
    public void update() {
        ScheduleTask scheduleTask = new ScheduleTask()
                .queryBuilder().taskName().finishBuilder()
                .letTaskName("TestTaskName")
            	.letTaskModule("TEST").letCronExpr("* * 1 * * ?");
        baseStaticDAO.update(scheduleTask);
    }
```

### 删除

```java
    /** 按设置的条件删除数据 */
    @Test
    public void delete() {
        ScheduleTask scheduleTask = new ScheduleTask()
                .letTaskName("TestTaskName");
        baseStaticDAO.delete(scheduleTask);
    }

	// 另有deleteByPrimaryKeys 按主键删除字段以及deleteBy直接按表名和字段值进行删除等API
```

### 计数

```java
    /** 按设置的条件统计数量 */
    @Test
    public void count() {
        ScheduleTask scheduleTask = new ScheduleTask()
                .letTaskName("TestTaskName");
        long total = baseStaticDAO.count(scheduleTask);
    }
```

## 2) 基于bean+operators的复杂操作

- 将不是等于的操作符如`>` `like` `in`等操作以`Operators` API展现，同时支持`and/or`条件关系

````java
    /** where TASK_MODULE='TOUCH_TASK' and TASK_NAME like '%TEST' */
    @Test
    public void testQueryOperators() {
        ScheduleTask scheduleTask = new ScheduleTask();
        scheduleTask.letTaskModule("TOUCH_TASK");
        scheduleTask.andOperators().name(ScheduleTask.Columns.TASK_NAME).like("%TEST");
        System.out.println(baseStaticDAO.queryOne(scheduleTask));
    }

    /** 内嵌式条件 where TASK_MODULE='TOUCH_TASK' and (TASK_NAME = 'hello' or TASK_NAME = 'TASK_TEST' */
    @Test
    public void testQueryNestOperators() {
        ScheduleTask scheduleTask = new ScheduleTask();
        scheduleTask.letTaskModule("TOUCH_TASK");
        scheduleTask.andNestOperators(
            new Operators()
            	.name(ScheduleTask.Columns.TASK_NAME).equal("hello")
                .or().equal("TASK_TEST")
            .finishNesting());
        System.out.println(baseStaticDAO.queryOne(scheduleTask));
    }

    /** where TASK_MODULE='TOUCH_TASK' and TASK_NAME in ('hello', 'TASK_TEST') */
    @Test
    public void testInOperators() {
        ScheduleTask scheduleTask = new ScheduleTask();
        scheduleTask.letTaskModule("TOUCH_TASK");
        scheduleTask.andOperators().name(ScheduleTask.Columns.TASK_NAME).in(new ArrayList<String>(){{add("hello"); add("TASK_TEST");}});
        System.out.println(baseStaticDAO.queryOne(scheduleTask));
    }
````

- Operators中支持的全量API见`org.walkframework.boot.ehdb.bean.operator.Comparator`

````java
public Operators greaterThan(Object value);
public Operators greaterOrEqualThan(Object value)
public Operators littleOrEqualThan(Object value);
public Operators littleThan(Object value);
public Operators in(Object value);
// 另有：orderBy/limit/offset等控制排序和分页，需按顺序注册操作
...
````

## 3) 基于SQL的操作

### 查询

- sqlId为sql_config表中的ID，表中配置SQL，修改之后需要清理缓存，而以sqlContent为参数的API则直接传入sql文本而不是ID，此类API适合在业务封装时使用，不需要依赖引用方的数据库配置

```java
/** 基于SQL内容和MAP响应的查询支持 */
List<Map<String, Object>> query(String sqlContent, Map<String, Object> params);

/** 通过sqlId进行无分页查询，最大返回结果依赖walk.ehdb.default-max-page-size，默认2000 */
<B extends BaseBean> List<B> queryBeansBySqlId(String sqlId, B params);

/**指定SQL查询单表分页数据，Bean为查询参数 */
<B extends BaseBean> PageResult<B> queryByPage(String sqlId, B query, Pagination page);

/**指定SQL查询单表分页数据，使用map等作为查询参数，指定返回类型 */
<B extends BaseBean> PageResult<B> queryByPage(String sqlId, Object query, 
                                               Pagination page, Class<B> clazz);
/** 指定SQL查询单表分页数据，Bean为查询参数 */
PageResult<Map<String, Object>> queryMapByPage(String sqlId, Object query, Pagination page);
// 其他查询SQL参见StaticDAO中的API
```

### 更新

- 更新SQL可以为update，也可以是delete

```java
/** 根据sqlId查询sql内容进行更新，更新参数可为bean或map */
boolean update(String sqlId, Object params);

/** 根据sqlId查询sql内容进行批量更新，更新参数可为bean或map的列表 */
boolean updateBatch(String sqlId, List<Object> params);

/** 直接根据sql内容进行更新，更新参数可为bean或map */
boolean updateBySql(String sqlContent, Object params);

// 其他更新SQL参见StaticDAO中的API
```

### 计数

```java
/** 通过指定sqlId和参数进行统计 */
long count(String sqlId, Object params);

/** 通过指定sql内容和参数进行统计 */
long countBySql(String sqlContent, Object params);
```

## 4) 动态SQL

配置sqlId或sql内容进行查询、更新等操作时，基于参数的不确定性，常需要使用动态SQL

### 语法糖

语法糖为基于标准SQL如`COLUMN_NAME = :COLUMN_VALUE`格式衍生的简化动态效果，目前内置的语法糖含两种

- 可选参数

````sql
-- 问号后缀模式，在标准操作符前后加空格的模式下使用，
where xx = :param1? and yy = :param2?
	如param1为null，则生成 where 1=1 and yy = :param2
	如param2为null，则生成 where xx = :param1
	如均为null, 则生成 where 1=1
````

- 列表参数

````sql
where column in :listParam
-- 默认listParam可空，不需要加问号后缀，空时无此参数查询
````

- 自定义语法糖，参考`org.walkframework.boot.ehdb.service.impl.InListParamSyntaxSugar`，继承`org.walkframework.boot.ehdb.service.AbstractSyntaxSugar`实现接口方法即可，可定义顺序，如果需要比其他某语法糖优先，顺序越小，优先级越高

### SPEL

- 使用spring el进行动态SQL渲染，使用缓存存储解析的语法，忽略性能影响

- 使用#{xxxx}包裹的内容为动态内容，内置部分函数供使用，如#isEmpty(参数名)/#nowByFormat('yyyy-MM-dd')/#sqlIn(列表参数)等

- 内置已注册函数全量可见：`org.walkframework.boot.util.CommonUtil`, `org.walkframework.boot.util.DateUtil`

- 可自行注册函数如下，逗号分隔多个静态工具类

  ````properties
  walk.springEL.functions=org.walkframework.boot.ehdb.util.DynamicSqlFunctions
  ````

- 使用参考

````sql
-- 配置column1必须等于column1参数值，日期字段小于今日，如果dynParam不为空则column2必须等于dynParam参数值的一个查询
select * from table_name where column1 = :column1 and date_column < #{#nowByFormat('yyyy-MM-dd')} #{#isEmpty(#root['dynParam']) ? '' : 'and column2 = :dynParam'}
````

### 更新历史

#### 3.6.0-2020/09/01

##### 特性变更

- updateBatch/updateBatchBySql开始将动态SQL视为最终一致SQL，只使用第一条记录去动态渲染最终SQL，以提高性能，不同态SQL需要人工拆分

##### 新增特性

- 新增updateBatchStepTransaction/updateBatchBySqlStepTransaction/saveBatchStepTransaction api，批量变更时强制使用事务编程保证批量提交，但是不保证事务最终一致，只是为了解决在事务覆盖不到不能手工声明事务时不再逐条提交的问题
- 新增动态表名抽取能力，在初始化bean时可使用setTableName(String tableName) api设置最终表名，也可在应用初始化时使用DynTableNameFactory.register api将某些bean的表名固定修改为其他名称，解决表名冲突或命名规则冲突问题


---
<div style="display: flex">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/walk-starter-base"><--基础依赖</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/walk-mq">消息队列--></a>
  </div>
</div>

{% endraw %}
