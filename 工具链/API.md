# Concrete API工具


```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-api-tools</artifactId>
            <!-- 建议在test作用于中生成api及手册 -->
            <scope>test</scope>
        </dependency>
```

```java
// generate(String desc, String path, String... packages) throws IOException
API.generate(渲染类型, 输出位置, 服务模块和ErrorCodes的包);
```

渲染类型: _服务提供类型_._类型_._使用者_._文档化格式_._版本_

    服务提供类型：JaxRS
    类型：code, doc
    使用者：backend, jquery, angularjs, angualr2, java, c#等
    文档化格式：gitbook, asciidoctor, markdown等


当前版本提供了 ~~7~~ 8 个API工具

| 渲染类型 | 作用 |
| --- | --- |
| JaxRS.doc.backend.gitbook.v1 | 生成服务模块的RESTFul API手册, Gitbook格式 |
| JaxRS.code.jquery.js.v1 | 生成基于jquery的javascript api |
| JaxRS.doc.jquery.gitbook.v1 | 生成jquery调用服务原子的API手册，Gitbook格式 |
| JaxRS.code.angular.ts.v1 | 生成ts的接口定义代码包，基于Observable模式 |
| JaxRS.code.angular.ts.v2 | 20180103，生成ts的接口定义代码包，基于Observable模式，推荐Angular版本高于4.3使用 |
| java.code.RxJava2.v1 | 根据ConcreteService转为RXService |
| WebSocket.code.jquery.js.v1 | 生成基于jquery的javascript api |
| WebSocket.code.angular.ts.v1 | 生成ts的接口定义代码包，基于Observable模式 |

------

- **2017-02-09**
    
    jaxrs.code.jquery.js.v1增加模块名属性。jaxrs.code.jquery.js.v1._moduleName_


