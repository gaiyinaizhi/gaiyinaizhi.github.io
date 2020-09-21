{% raw %}
# 过滤器

过滤器用于对源表达式进行变形、转换、缩减等，一般为在表达式后加`| filterName(filterParam)`格式，参数部分可省略，简化为`| filterName`格式

过滤器大体分为：格式化、循环、事件几类

## 格式化过滤器

- `uppercase`    单词大写

  ```html
  <span>{{someText | uppercase}}</span>
  ```

- `lowercase`      单词小写

  ````html
  <span>{{someText | lowercase}}</span>
  ````

- `ellipsis`           超长截断，默认追加`...`

  ````html
  <span>{{someText | ellipsis(20, '---')}}</span>  // 超出20位截断20个字符并追加---，最后参数可省略，使用默认...
  ````

- `date`               时间类型格式化

  ````html
  <span>{{dateParam | date('yyyy-MM-dd hh:mm:ss')}}</span> <!-- 默认yyyy-MM-dd hh:mm:ss格式，可加入参定制 --> 
  ````

- `decimal`         小数位数格式化

  ````html
  <span>{{digitalParam | decimal(2)}}</span> // 两位小数
  ````

- 知识点
  - 其他指令中也可以使用，如`w-text`, `w-attr`, `w-value`等

  

## 数组/对象过滤器

- `filterBy` 过滤数组或对象

  - 过滤数组

    ````html
    <!-- 简单使用：过滤数组中顺序大于0并且对应行数组中id不为1的行数据 -->
    <div :for="(index, item) in somArray | filterBy('index > 0 && value.id != 1')">
    <!-- 如果过滤时使用的字段不只是index/value中的值，则使用函数作为过滤依据 -->
    <div :for="(index, item) in somArray | filterBy(filterFuncName)">
    <script>
      var vm = walkvm('id', {
        filterFuncName: function(value, index) {
          return vm.somValue && index > 0;
        } 
      })
    </script>
    ````

  - 过滤对象

    ````html
    //过滤对象中key不等于key1并且value中prop1的值不为propValue的其他所有key/value进行循环，常用来循环展示属性类的对象如：{'commName':'商品名称', 'commPrice': 12}
    <div :for="(key, value) in jsonObject | filterBy(key != 'key1 && value.prop1 != 'propValue')">
    ````

- `orderBy` 排序数组或对象

  ```html
  <!-- 指定字符串作为item中的排序变量名 -->
  <div :for="(index, item) in somArray | orderBy('KEY1')">
  <!-- 指定字符串作为item中的排序变量名，并且指定排序算法 -->
   <div :for="(index, item) in somArray | orderBy('KEY1', orderByFunc)">
  ```

- `limit` 数组长度限制

  ````html
  <div :for="(index, item) in somArray | limit(5)"> // 抽取数组中前5条数据 
  <div :for="(index, item) in somArray | limit(1, 5)"> // 抽取数组中第2-5条数据
  ````

- `asArray`      数字转数组

  常用于一个简单循环，为了重复固定次数做某些操作

  ````html
  <div :for="(index, item) in ditialParam | asArray"> //循环ditialParam次
      <span>{{'current index is: ' + index}}</span> 
  </div>
  ````

  

## 事件过滤器

- `prevent` 阻止默认响应

- `stop` 阻止冒泡

- `when` 当真值表达式为true时才执行事件响应

  ````html
  <a @click="someFunc | when(clickedTimes < 2)">click to reponse</a>
  ````

---
<div style="display: flex">
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walkvm/basic/for"><--循环</a>
  </div>
  <div style="display: flex;flex:1;align-items: center;">
    <a href="https://gaiyinaizhi.github.io/walkvm/index">回列表</a>
  </div>
</div>

{% endraw %}