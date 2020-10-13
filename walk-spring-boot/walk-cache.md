{% raw %}

# walk-cache 缓存组件使用说明

## 能力综述

`walk-cache`组件桥接了spring-data的CacheManager和Cache的使用，在原有的配置上进行扩充，声明式使用对象缓存还是字符串缓存，
并且提供了CacheLoader.get(key, loader)方式进行动态资源加载判定，支持指定缓存时间及随机波动能力

## 使用说明

- 使用`@WCache`注解进行初始化`ICache`对象，使用ICache可获得基于spring Cache对象扩展的所有能力
- 配置默认的缓存失效时间，配置波动范围
- 使用StringCacheLoader，提供`get(cache, key, typeReference, loader)`相关api进行加载
- 使用StringCacheLoader，提供put/remove/exists api进行添加、删除、校验操作
- 使用分布式定时任务进行缓存定时加载，保证热点数据不失效，定时器可根据活动时间等在线调整


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