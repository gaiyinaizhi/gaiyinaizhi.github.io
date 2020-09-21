{% raw %}
# 指令

## 全部指令

- 在浏览器控制台使用api `walkvm.printDirs()`打印全部指令及扫描优先级
- 指令列表如下：

| 指令                | 描述                                                         | 优先级 |
| ------------------- | ------------------------------------------------------------ | ------ |
| controller          | 指定一个扫描区域，用于生成一个vm对象                         | 9900   |
| if                  | 真值表达式进行判断，用于校验该dom是否渲染                    | 10     |
| for                 | 循环进行dom渲染                                              | 20     |
| noop                | 为了声明一个监听做动态响应，和watch相似，定义在dom上为了就近说明和dom的展示相关 | 30     |
| attr                | 声明一个属性，非指令关键字时可以使用短指令实现，`w-attr-title="hello"`等价于`:title="hello"` | 9700   |
| class               | 使用变量或表达式来指定class属性                              | 9900   |
| css                 | 声明一个变量来代表样式信息，具体表现和jquery的css用法相似    | 9900   |
| display             | 声明如何展示一个dom，通过变量或表达式来确认是show还是hide    | 10000  |
| expr                | 插值表达式，即使用`{{xxx}}`直接插入到html中的文本            | 10100  |
| html                | 插入一段html文本                                             | 10400  |
| text                | 插入一段纯文本，html不会被解析而是直接展示                   | 11600  |
| on                  | 事件监听，`w-on-click="clickFunc"`等价于`@click="clickFunc"` | 11100  |
| value               | 声明一个dom的value值                                         | 11800  |
| toggleattr/attrif   | 真值表达式来确定是否标记某属性                               | 11600  |
| toggleclass/classif | 真值表达式来确定是否增加某class，例：`w-classif-hidden="isShow"` | 11600  |
| duplex              | 双向绑定，通常用于绑定form表单中某input/select的值           | 99999  |
| jq                  | 桥接jquery的api，不重要                                      | 10600  |

---

<div style="display: flex">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walkvm/basic/index"><--入门</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walkvm/index">回列表</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walkvm/basic/directives">插值表达式--></a>
  </div>
</div>

{% endraw %}