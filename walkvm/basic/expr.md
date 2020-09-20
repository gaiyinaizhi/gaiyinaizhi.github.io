# 插值表达式

位于文本节点中的双重花括号`{{}}`，当然这个可以配置。此指令为文本`w-text`指令的简单形式。

```html
<body w-controller="test">
    
    <p>{{@aaa}}{{@bbb}} 这个性能差些</p>
    <p>{{@aaa+@bbb}} 这个性能好些</p>
    <p>{{@aaa+@bbb  | uppercase}} 过滤器必须放在表达值的后端</p>
</body>
<script>
    walkvm.define({
        $id: 'test',
        aaa: 'aaa',
        bbb: 'bbb'
    })

</script>
```

知识点：

-  一个文本节点里尽量使用一个插值表达式
- 表达式中可以做连接、运算等操作
- 表达式最后可增加过滤器，详情见`过滤器`章节

## 
| [<-指令集](https://gaiyinaizhi.github.io/walkvm/basic/directives)                | [回列表](https://gaiyinaizhi.github.io/walkvm/index)                                                         | [循环->](https://gaiyinaizhi.github.io/walkvm/basic/for) |
| ------------------- | ------------------------------------------------------------ | ------ |
