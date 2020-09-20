# 入门

## walkvm

walkvm是一款基于虚拟DOM与属性劫持的 迷你、 易用的前端MVVM框架， 拥有超优秀的兼容性, 支持移动开发, 无需编译, 开箱即用

本框架中的vm是一种利用Proxy或 Object.defineProperties或VBScript创建(`低版本浏览器兼容时使用`)的特殊对象。

## 示例

````html
<!DOCTYPE html>
<html>
    <head>
        <title>first example</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <script src="./dist/walkvm.js"></script>
    </head>

    <body>
        <div id="testController" w-controller>
          <p>for循环输出：</p>
          <ul>
            <li w-for="item in list|limit(3)" :title="item.text">
                {{item.text|ellipsis(30)}}
            </li>
          </ul>
          <form>
            <div class="form-group mt-2">
              <label for="exampleInputEmail1">双向绑定</label>
              <input type="text" class="form-control" placeholder="可输入任意字符" :duplex-input="text">
              <small class="form-text text-muted mb-2">已输入 [{{text}}]</small>
              <input type="text" class="form-control" placeholder="只能输入8位数字" data-filter="number,length" data-filter-length="8" :duplex="number">
              <small class="form-text text-muted">已输入 [{{number}}]</small>
            </div>
            <button type="submit" class="btn btn-primary">Submit</button>
          </form>
        </div>
    </body>
    <script>
            var vm = walkvm('testController', {
                  list: [
                    {text: '只显示3条'},
                    {text: '因为使用了过滤器limit'},
                    {text: '超过30字后会截断，因为使用了文字过滤器ellipsis，所以后面的显示不到了'},
                    {text: '测试文字4'},
                    {text: '测试文字5'}
                  ],
                  text: '',
                  number: ''
                });                    
        </script>
</html>
````



知识点简要说明：

- 使用walkvm或walkvm.define生成vm对象，首参传入controller的id，或在大对象中使用$id指定
- 指令以`w-duplex`形式或使用短指令`:duplex`形式，或以花括号包围的插值表达式`{{number}}`形式
- 引导符，以`@`字符来告诉框架这些变量是来自vm的
- 定义时将自动扫描对应的`id`属性或者以`w-controller`定义的对应id所在dom区域

## 
|                 | [回列表](https://gaiyinaizhi.github.io/walkvm/index)  | [指令集->](https://gaiyinaizhi.github.io/walkvm/basic/directives) |
| ------------------- | ------------------------------------------------------------ | ------ |
