# concrete 配置规范

concrete在资源文件的使用上，有一套规范：`TAG.MODULE.key`，配置该属性时，优先级如下：

- `TAG`.`MODULE`.properties 中的 `key`
- `TAG`.properties 中的 `MODULE`.`key`
- concrete.properties 中的 `TAG.MODULE.key`

## 使用

```java
String value = ConcreteHelper.getString(tag, module, key);
```


## concrete的一些配置

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