# Java客户端调用

Concrete 工具链默认提供了一下两个实现。
```xml
    <dependency>
        <groupId>cc.coodex</groupId>
        <artifactId>concrete-jaxrs-serializer-fastjson</artifactId>
    </dependency>

    <dependency>
        <groupId>cc.coodex</groupId>
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

一样的，JavaClient也支持数据模拟，即，无server并行开发，增加java参数`-Dcc.coodex.concrete.jaxrs.devMode=true`

-----
TODO: 暂时还不支持ssl，需要考虑切入点和接入方式