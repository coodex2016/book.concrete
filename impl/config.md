# concrete 配置规范

concrete在资源文件的使用上，有一套规范：`TAG.MODULE.key`，配置该属性时，优先级如下：

- `TAG`.`MODULE`.properties 中的 `key`
- `TAG`.properties 中的 `MODULE`.`key`
- concrete.properties 中的 `TAG.MODULE.key`

## 使用

```java
String value = ConcreteHelper.getString(tag, module, key);
```


### concrete-api-tools

- tag: api_gen， `API.genreteFor`使用到
- module: 由`API.genreteFor`的第一个参数传入
- keys
  - desc: [渲染描述符](../impl/API.md)
  - path: 生成的根路径

### conrete-client

- tag: client
- module: 由`Client.getInstance`接口的第二个参数传入，或者`@ConcreteClient`的参数传入
- keys
  - location: 关键的，被调用服务发布的location
    - http:// or https://， concrete-jaxrs-client支持
    - local: 本地调用，即在依赖注入容器中调用该服务的实现
    - ws:// or wss://，concrete-websocket-client-rx支持
    - dubbo: concrete-dubbo-client支持
  - tokenManagerKey: 多个服务调用者之间共享token默认是根据`module`来共享，我们也可以指定该属性，使相同的tokenManagerKey的服务调用者共享同一个token,当然，前提是，发布的服务是相同的token管理
  - tokenTransfer: 模块到模块间调用时，是否将调用者的token传递到被调用者端，例如 前端 --> B --> C，模块C也需要校验token，那么，B调用C时，应该指定tokenTransfer为true
  - 其他扩展
    - dubbo扩展
      - registry: 多个用`，`分隔的dubbo注册中心地址
      - name: 对应dubbo的Application名
      
## concrete的一些配置

### concrete.properties

| 配置名 | 说明 |
| ----- | ---- |
| messagePattern.resourceBundles | 用来指定默认情况下，错误信息模板使用的bundles，使用`,`分隔多个bundle |
| service.executor.corePoolSize | concrete服务线程池，最低线程数，默认0 |
| service.executor.maximumPoolSize | concrete服务线程池，最大线程数，默认Integer.MAX_VALUE |
| service.executor.keepAliveTime | concrete服务线程池，当线程空闲达到该阈值时会被回收，单位，秒，默认60 |
| aspect.bean.validation | 是否开启beanValidation，早期参数，默认为true，不推荐使用 |
| concrete.api.packages |   |
| concrete.appSet |  |
| token.maxIdleTime |  分钟 |
| tokenBasedTopicMessage.cacheLife |   |
| zipkin.location |  |
| module.name |  |
| queue.default |  |
| counter.thread.pool.size |  |
| jsr339.charset | 默认utf-8 |


