## 服务跟踪

### 概述

- 提供统一的服务跟踪能力，可以在界面通过提示的`traceId`到后台查询日志而不是去代码里找关键字进行过滤
- 提供关键服务出入参数入库能力，可以直接在`分库存储数据库`中查询关键服务的出入参，避免去多台主机上捞日志的麻烦

### 服务跟踪配置

- 通过`walk_trace_config`表进行关键服务配置

  | 关键字段名   | 字段描述                                                     |
  | ------------ | ------------------------------------------------------------ |
  | module       | 配置归属模块                                                 |
  | service_name | 服务名称配置                                                 |
  | service_desc | 服务名称描述                                                 |
  | on           | 1:打开,0关闭                                                 |
  | condition    | 过滤表达式，如果配置了，可以使用此方法来过滤只记录某些参数下的日志，如：`param1 == 'userValidate' and hasError` 过滤该服务第一个参数值为`userValidate`并且服务有异常的 |
  | trace_type   | S:service only只记录当前服务, C: chain记录整个调用链         |
  | param_enable | 1:打开入参记录, 0: 关闭         |
  | output_enable| 1:打开输出参数记录, 0: 关闭         |
  | log_days     | 日志保存时长         |

- 组件默认会记录报错的服务调用，并且是记录整个调用链，如果需要修改需要通过新增服务跟踪配置进行覆盖

- 记录的日志到`分库数据库`中，并且以`trace_id`分库

- 服务可跟踪`rpc`远程调用类型的远程调用


### 跟踪日志

| 字段名       | 字段描述                                   |
| ------------ | ------------------------------------------ |
| trace_id     | 跟踪ID                                     |
| service_name | 服务名称                                   |
| module       | 归属模块                                   |
| input        | 入参                                       |
| output       | 出参                                       |
| has_error    | 1:异常，0:正常                             |
| exception    | 异常信息描述，压缩的描述信息，简化类的展示 |
| start_time   | 执行开始时间                               |
| end_time     | 执行结束时间                               |
| host_ip      | 服务执行的主机IP                           |

## 
| [<-动态测试桩](https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-mock) | [回列表](https://gaiyinaizhi.github.io/walk-spring-boot/index) | [服务限流->](https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-tools-limit) |
| ------------------- | ------------------------------------------------------------ | ------ |
