# for循环

for 指令可用于循环对象中的key/value进行输出，或输出数组中的行数据输出渲染

````html
<div :for="(index, item) in somArray"> // 正常循环全部数组进行渲染
    <span>{{'current index is: ' + index + ', value is: ' + item.value}}</span> 
</div>
<div :for="(index, item) in somArray | limit(5)"> // 使用过滤器限制只渲染前5条
    <span>{{'current index is: ' + index + ', value is: ' + item.value}}</span> 
</div>
<div :for="(index, item) in somArray | filterBy('index > 10')"> // 使用过滤器限制只渲染index大于10的记录
    <span>{{'current index is: ' + index + ', value is: ' + item.value}}</span> 
</div>
````

知识点：

- 循环对象可为对象、数组及类数组的对象如arguments/DomElements等

- 使用过滤器对数组进行处理后输出，详情参见`过滤器`章节


## 
| [<-插值表达式](https://gaiyinaizhi.github.io/walkvm/basic/expr)                | [回列表](https://gaiyinaizhi.github.io/walkvm/index)                                                         | [过滤器->](https://gaiyinaizhi.github.io/walkvm/basic/filter) |
| ------------------- | ------------------------------------------------------------ | ------ |
