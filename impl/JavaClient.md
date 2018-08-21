# Java客户端调用

> 2017-09-01 增加RXJava2的支持，往下翻

Concrete 工具链默认提供了以下~~两个~~实现，如未指定invoker则使用JaxRS Client。

建议如下使用
```xml
    <dependency>
        <groupId>org.coodex</groupId>
        <artifactId>concrete-jaxrs-client</artifactId>
    </dependency>
    
    <!-- 使用jersey作为jaxrs client的实现 -->
    <dependency>
        <groupId>org.glassfish.jersey.core</groupId>
        <artifactId>jersey-client</artifactId>
    </dependency>

```

【已废弃】
```xml
    <dependency>
        <groupId>org.coodex</groupId>
        <artifactId>concrete-jaxrs-invoker-okhttp3</artifactId>
    </dependency>
```

concrete.properties配置

    # 默认utf-8
    # global
    jaxrs.client.charset = 
    # or
    jaxrs.client.charset.domain =

    concrete.client.domain = 
    # 或者
    concrete.client._server_.domain = 


```java
    ServiceExample serviceExample;
    
    // 使用concrete.client.domain
    serviceExample = Client.getInstance(ServiceExample.class);

    // 使用concrete.client.server.domain
    serviceExample = Client.getInstance(ServiceExample.class, "server");

    // 使用 http://serverName
    serviceExample = Client.getInstance(ServiceExample.class, "http://serverName");
    
    // 使用 http://serverName 并且指定Global_Token为token的作用域，该值非空相同的，共享token
    serviceExample = Client.getInstance(ServiceExample.class, "http://serverName", "Global_Token");
```

一样的，JavaClient也支持数据模拟，即，无server并行开发，增加java参数`-Dorg.coodex.concrete.jaxrs.devMode=true`

如果domain为local，则直接调用本地服务

## RXClient

concrete.properties配置



### jaxrs client to rx client

```properties
## domain identify
concrete.client.domain=
## or
concrete.client._server_.domain=

## identify的调用类型，为后续jms等异步服务预留
concrete.client.type=
## or
concrete.client._server_.type=

## 调用identify是否异步，默认为true
concrete.client.async=
##
concrete.client._server_.async=


```

1. 增加依赖
```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs-client-rx</artifactId>
        </dependency>
```

2. 生成RX api
```java
        API.generate(ReactiveStreamsRender.RENDER_NAME,
                apiRootPath,
                ServiceExample.class.getPackage().getName());
```

3. 使用
```java
    ServiceExample_RX rx = RXClient.getInstance(ServiceExample_RX.class, domain);

    rx.all().subscribe(new Observer<List<Book>>() {
        @Override
        public void onSubscribe(Disposable d) {
        }

        @Override
        public void onNext(List<Book> books) {
            for (Book book : books) {
                System.out.println(book);
            }
        }

        @Override
        public void onError(Throwable e) {
            e.printStackTrace();
        }

        @Override
        public void onComplete() {
            System.out.println("complete");
        }
    });
```



## okHttp3配置【已废弃】
okHttp3.properties
    
    # 所有可被访问的域需要在此列出
    _server_ = 
    # 2017.03.27 sslContext移除
    # _server_.ssl =  

## 2017-03-09

- 修复BigString注解的全部缺陷
- JavaClient模块强化，增加AOP功能
    - 继承org.coodex.concrete.core.intercept.AbstractInterceptor，重载有关方法
    - META-INF/services/org.coodex.concrete.core.intercept.ConcreteInterceptor 中增加需要使用到的拦截器
    - org.coodex.concrete.jaxrs.Client.getUnitFromContext：根据拦截器上下文获取Unit信息