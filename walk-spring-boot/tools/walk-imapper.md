## IMAPPER快速参数映射及服务编排

### 概述

imapper报文转换旨在将界面入参和会话参数等自动转换为外部服务调用需要的入参格式

服务编排能力旨在将一次复杂的外部调用使用xml配置进行简化和清晰化

提供动态发布能力，实现在线更新应用的能力

### 依赖

```xml
<dependency>
    <groupId>org.walkframework.boot</groupId>
    <artifactId>walk-starter-base</artifactId>
    <version>${latestVersion!1.9.6}</version>
</dependency>

<dependency>
    <groupId>org.walkframework.boot</groupId>
    <artifactId>walk-tools-imapper</artifactId>
    <version>${latestVersion!1.7.7}</version>
</dependency>
```

- 特别说明，如果不需要使用redis订阅能力，可以在`walk-tools-imapper`依赖中增加如下排除语法取消

```xml
    <exclusions>
        <exclusion>
            <groupId>org.walkframework.boot</groupId>
            <artifactId>walk-tools-config</artifactId>
        </exclusion>
    </exclusions>
```

### 报文转换

- 如下示例
- xml中的节点结构代表最终生成的数据格式，叶子节点中的文本代表选择表达式，获得取值来源
- 提供object/list/类型转换，默认prototype原型，即来源字段的类型代表当前节点的类型
- 提供format功能，将数据进行类型转换或trim等操作
- 提供validate功能，校验必传，非空等
- 提供default功能，提供字段默认能力，另有defaultType配置配合使用，达到对default值做特殊处理的效果
- 提供transform功能，将字段取值进行变形操作，如：0,1变形为男，女，支持spel表达式处理，可进行简单数据操作或复杂的服务调用等
    - transform使用spring el进行数据抽取和转换，通常_root指向根入参对象，this模式时_root指向外部对象指定的src对象，通过_root._root可取到根入参
    - 使用@xxService.xx(非空参数名, _root['可空的参数名']， #safeGet(_root, '路径1.路径2.路径3等参数安全获取'))
- 提供enumType/enumMap功能，提供将字段做枚举类型转换的能力，[详见atom.xsd属性定义](https://gaiyinaizhi.github.io/walk-spring-boot/tools/imapper/atom.xsd)
- 提供holder主叶子校验功能，当主叶子值为空时整个节点不做转换
- 提供filter功能，过滤对象或列表类型节点，使不符合条件的对象被过滤掉
- 提供returnType注册当前节点代表的java bean对象功能
- 提供append-mode追加模式，直接将来源对象全量复制，可配置更多配置或同名字段用于追加或替换原值
- 提供`prepareDefaultTransformers`配置预加载内置变量如序列、时间戳等
- 提供this模式加载，对象下简单字段直接通过直接字段名获取而不是对象的别名`var`配置，批量this字段时大幅度提高性能
    - this模式下，如果是直接字段会都认为是当前父级src对应的字段取值，如果是xx.xx这种表达式，仍然从根对象下开始查询
    - this模式下，执行transform转换时，_root指向当前对象(当外层有个src="xx"指向某一对象时)，_root._root指向整体入参，非this模式时，_root即为根对象
- 更多功能直接参考[atom.xsd属性定义](https://gaiyinaizhi.github.io/walk-spring-boot/tools/imapper/atom.xsd)。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<imapper thisMode="true" prepareDefaultTransformers="SYSDATE19" xmlns="http://www.walkframework.com/imapper/atom">
    <ORDER_REQ>
        <ORDER src="INPUT_ORDER" transform="@xmlMapperUseCaseService.appendParam(_this['ORDER_ID'], _root[INPUT_ORDER_SUB_ITEM][], #safeGet(_root, 'INPUT_ORDER_GOODS.xxx.TMPL_ID'))" var="order">
            <ORDER_ID holder="1" format="trimToNull">ORDER_ID</ORDER_ID>
            <MORE_INFO>moreOrderInfo</MORE_INFO>
            <SOME_CODE default="9521">SOME_CODE</SOME_CODE>
            <SEQ default="SEQ" defaultType="TRANSFORMER"></SEQ>
            <PROTOCAL src="INPUT_ORDER_PROTOCOL" var="protocal">
                <PROTOCOL_ID holder="1">PROTOCOL_ID</PROTOCOL_ID>
                <UPDATE_TIME>_root.SYSDATE19</UPDATE_TIME>
                <PROVINCE_CODE enumType="PROVINCE_CODE">PROVINCE_CODE</PROVINCE_CODE>
            </PROTOCAL>
            <CUST_INFO src="INPUT_ORDER_SUB_ITEM" var="custInfo">
                <PSPT_ID holder="1">PSPT_NO</PSPT_ID>
                <CUST_ID>INPUT_ORDER.CUST_ID</CUST_ID>
                <CUST_NAME default="客户">CUST_NAME</CUST_NAME>
                <CUST_TYPE enumType="CUST_TYPE">CUST_TYPE</CUST_TYPE>
                <SEX enumMap="1=男, 0=女">SEX</SEX>
                <BIRTHDAY>BIRTHDAY</BIRTHDAY>
                <OTHER_DATE defaultType="SELECTOR" default="_root.SYSDATE19">OPERATE_TIME</OTHER_DATE>
            </CUST_INFO>
        </ORDER>
    </ORDER_REQ>
</imapper>
```

### 流程编排

- 如下示例

- `flow`定义一个流程调用，可以执行`service`，也可以执行`state-resolver`进行流程状态判断，也可以同时存在，使用`state-resolver`对外部服务调用进行状态判断

- `output`定义一个输出节点，一般分为正常输出、业务失败输出和服务异常输出，声明`exception="true"`即表示异常输出节点

- 输出节点的子节点可认为是一次报文转换，可使用报文转换中的所有特性

- 更多功能直接参考[flow.xsd](https://gaiyinaizhi.github.io/walk-spring-boot/tools/imapper/flow.xsd)


```xml
<?xml version="1.0" encoding="utf-8"?>
<imapper type="flow" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.walkframework.com/imapper/flow">

    <!--参数预处理-->
    <flow id="paramDeal" state-resolver="xxService.dealXxParam(_root)">
        <transition on="firstLevel" to="firstLevel"></transition>
        <transition to="secondLevel"></transition><!-- 没有on则表示最后的else，全量匹配 -->
    </flow>

    <!--一级操作，只使用state-resolver返回响应码处理的情况，transition>on配置只处理响应状态码-->
    <flow id="firstLevel" state-resolver="xxService.getFirstLevel(_root)">
        <transition on="success" to="secondLevel"></transition>
        <transition to="callFail"></transition>
    </flow>

    <!--二级操作，调用服务后再使用state-resolver返回响应码处理的情况，_this代表服务结果，_root代表上下文-->
    <flow id="secondLevel" service="xxService.call('secondLevel',_root)" state-resolver="xxDealService.dealSecondRsp(_this,_root)">
        <transition on="success" to="thirdLevel"></transition>
        <transition to="callFail"></transition>
    </flow>

    <!--三级操作，调用服务后直接通过transition表达式判断服务响应的结果再做transition，#root代表返回值-->
    <flow id="thirdLevel" service="xxService.call('thirdLevel',_root)">
        <transition on="#root[respCode] == '0000'" to="callSuccess"></transition>
        <transition to="callFail">
            <!-- 设置错误码到上下文 -->
            <param name="respCode" default="1234"></param>
        </transition>
    </flow>

    <!--业务调用失败，返回失败信息-->
    <output id="callFail" type="ResponseInfo">
        <respData>respData</respData>
        <respMsg>respMsg</respMsg>
        <respCode>respCode</respCode>
    </output>


    <!--调用成功，返回成功信息-->
    <output id="callSuccess" type="ResponseInfo" > <!-- 调用成功，正常返回 -->
        <respData>respData</respData>
        <respMsg default="xx成功！">respMsg</respMsg>
        <respCode default="0000">respCode</respCode>
    </output>

    <!--调用异常，返回错误信息，且 当前流程中有且只有一个异常处理-->
    <output id="failOutput" exception="true">
        <respCode default="8888">errorCode</respCode>
        <respMsg default="操作成功">exception-msg</respMsg>
        <respData>respData</respData>
    </output>


</imapper>
```

---
<div style="display: flex;font-size: 14px">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/walk-scheduler"><--分布式定时任务</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walk-spring-boot/tools/walk-mock">动态测试桩--></a>
  </div>
</div>
