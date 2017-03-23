# Java客户端调用

Concrete 工具链默认提供了以下两个实现。
```xml
    <dependency>
        <groupId>org.coodex</groupId>
        <artifactId>concrete-jaxrs-serializer-fastjson</artifactId>
    </dependency>

    <dependency>
        <groupId>org.coodex</groupId>
        <artifactId>concrete-jaxrs-invoker-okhttp3</artifactId>
    </dependency>
```

concrete.properties配置

    concrete.serviceRoot = 
    # 或者
    concrete._server_.serviceRoot = 


```java
    ServiceExample serviceExample;
    
    // 使用concrete.serviceRoot
    serviceExample = Client.getBean(ServiceExample.class);

    // 使用concrete.server.serviceRoot
    serviceExample = Client.getBean(ServiceExample.class, "server");

    // 使用 http://serverName
    serviceExample = Client.getBean(ServiceExample.class, "http://serverName");
```

一样的，JavaClient也支持数据模拟，即，无server并行开发，增加java参数`-Dorg.coodex.concrete.jaxrs.devMode=true`

如果serviceRoot为local，则直接调用本地服务

okHttp3.properties
    
    # 所有可被访问的域需要在此列出
    _server_ = 
    _server_.ssl = 

## 2017-03-09

- 修复BigString注解的全部缺陷
- JavaClient模块强化，增加AOP功能
    - 继承org.coodex.concrete.core.intercept.AbstractInterceptor，重载有关方法
    - META-INF/services/org.coodex.concrete.core.intercept.ConcreteInterceptor 中增加需要使用到的拦截器
    - org.coodex.concrete.jaxrs.Client.getUnitFromContext：根据拦截器上下文获取Unit信息