## IMAPPER快速参数映射及服务编排

### 概述

imapper报文转换旨在将界面入参和会话参数等自动转换为外部服务调用需要的入参格式

服务编排能力旨在将一次复杂的外部调用使用xml配置进行简化和清晰化

提供动态发布能力，实现在线更新应用的能力

### 报文转换

- 如下示例
- xml中的节点结构代表最终生成的数据格式，叶子节点中的文本代表选择表达式，获得取值来源
- 提供object/list/类型转换，默认prototype原型，即来源字段的类型代表当前节点的类型
- 提供format功能，将数据进行类型转换或trim等操作
- 提供validate功能，校验必传，非空等
- 提供default功能，提供字段默认能力
- 提供transform功能，将字段取值进行变形操作，如：0,1变形为男，女，支持spel表达式处理，可进行简单数据操作或复杂的服务调用等

```xml
<?xml version="1.0" encoding="utf-8"?>
<imapper>
    <req>
        <params>
            <xxReq>
                <order type="object">
                    <param1 transform="@xxDealService.getParam1(_root)"></param1>
                    <param2 default="">object.param2</param2>
                    <param3 format="trimToNull">param3</param3>
                    <param4 type="list"
                               transform="@xxSubmitService.composeXxItem(_root)" var="xxItem">
                        <param5>object2.param5</param5>
                        <param6 default="2">param6</param6>
                        <param7 type="list" src="xxItem.feeInfo" var="xxInfo">
                            <param8 default="">xxInfo.feeMode</param8>
                        </param7>
                    </param4>
                </order>
            </xxReq>
        </params>
    </req>
</imapper>
```

### 流程编排

- 如下示例
- `flow`定义一个流程调用，可以执行`service`，也可以执行`state-resolver`进行流程状态判断，也可以同时存在，使用`state-resolver`对外部服务调用进行状态判断
- `output`定义一个输出节点，一般分为正常输出、业务失败输出和服务异常输出，声明`exception="true"`即表示异常输出节点
- 输出节点的子节点可认为是一次报文转换，可使用报文转换中的所有特性

```xml
<?xml version="1.0" encoding="utf-8"?>
<imapper type="flow" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.walkframework.com/imapper/flow">

    <!--参数预处理-->
    <flow id="paramDeal" state-resolver="xxService.dealXxParam(_root)">
        <transition on="firstLevel" to="firstLevel"></transition>
        <transition to="secondLevel"></transition>
    </flow>

    <!--一级操作-->
    <flow id="firstLevel" state-resolver="xxService.getFirstLevel(_root)">
        <transition on="success" to="secondLevel"></transition>
        <transition to="callFail"></transition>
    </flow>

    <flow id="secondLevel" service="xxService.call('secondLevel',_root)" state-resolver="xxDealService.dealSecondRsp(_this,_root)">
        <transition on="success" to="callSuccess"></transition>
        <transition to="callFail"></transition>
    </flow>

    <!--业务调用失败，返回失败信息-->
    <output id="callFail" type="ResponseInfo">
        <respData>respData</respData>
        <respMsg>respMsg</respMsg>
        <respCode>respCode</respCode>
    </output>


    <!--调研成功，返回成功信息-->
    <output id="callSuccess" type =  "com.chinaunicom.cbss2.beeaction.bean.biz.BeeActionResponse" > <!-- 调用成功，正常返回 -->
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
<div style="display: flex">
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