# 2.2 java端调用服务

`concrete-client` 提供了规范的java端(1.6以上)的调用方式，并通过插件对`jaxrs`,`websocket`，`dubbo`发布的服务进行同步、异步调用支持，[详见](../impl/JavaClient.md)

本例中，我们使用`jaxrs`的同步调用方式。

## `demo-client-invoker`

我们建一个`demo-client-invoker`模块来演示相关功能，一样的，目录为`demo/client-invoker`

pom中，增加如下依赖

```xml
    <dependencies>

        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>demo-api</artifactId>
            <version>${project.parent.version}</version>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.glassfish.jersey.media</groupId>
            <artifactId>jersey-media-json-jackson</artifactId>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>

    </dependencies>
```

- 因为我们要按照api规范调用，所以依赖`demo-api`
- 因为服务端采用`jaxrs`发布，所以依赖`concrete-jaxrs-client`
- 依赖`jersey-client`作为jaxrs规范的实现
- 依赖`jersey-media-json-jackson`作为json的序列化和反序列化的工具
- 依赖`slf4j-log4j12`，方便查看调用过程

建一个类来演示代码 `org.coodex.concrete.demo.invoker.ClientExample`

```java
package org.coodex.concrete.demo.invoker;

import org.coodex.concrete.Client;
import org.coodex.concrete.demo.api.DemoService;

public class ClientExample {

    public static void main(String [] args){
        DemoService demoService = Client.getInstance(DemoService.class, "demo");
        System.out.println(demoService.sayHello("davidoff"));
    }
}
```

`org.coodex.concrete.Client`也是使用的concrete[配置规范](../impl/config.md#conrete-client),我们配置一个`client.demo.properties`

client.demo.properties
```properties
location=http://localhost:8080/jaxrs
```

> `concrete-jaxrs-client`对location是http(s)协议的进行支持

log4j.properties
```
log4j.rootLogger=debug, console
log4j.logger.org.springframework=warn, console
log4j.logger.org.coodex.pojomocker=warn, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss} [%l]%n[%p] %m%n%n
```

把`demo-release`的服务跑起来，然后run，观察一下控制台

下一步，我们会演示在[依赖注入环境中如何使用Client](step2_3.md)，需要修改一下已有的代码。

截止到目前的代码 https://github.com/coodex2016/concrete-demo/tree/step2_2