### cocnrete-bom

concrete的物料清单，整合了基于concrete开发所需要的大部分artifact，方便使用。

包括：

- concrete全套: {{book.concreteVersion}}
- spring-framework: 4.3.10.RELEASE
- spring-data: Ingalls-SR6
- spring-boot: 1.5.6.RELEASE
- jersey: 2.25.1
- dubbo: 2.6.1
- freemarker: 2.3.23
- fastjson: 1.2.32
- javassist: 3.22.0-GA
- brave: 5.1.3
- hibernate: 5.2.8.Final
- hibernate-valdator: 5.4.0.Final
- druid: 1.1.2
- acrivemq-client: 5.15.4
- rabbitmq-client: 5.3.0
- slf4j: 1.7.25
- el-impl: 2.2

使用方法，在pom.xml中定义concrete.version为 {{book.concreteVersion}}，在`dependencyManagement`中的`depenencies`加入一个以下的片段
```xml
    <dependency>
        <groupId>org.coodex</groupId>
        <artifactId>concrete-bom</artifactId>
        <version>${concrete.version}</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
```

### concrete-api

定义接口的规范，由模块中的api模块依赖，作用域推荐设为`provided`

```xml
    <dependency>
        <groupId>org.coodex</groupId>
        <artifactId>concrete-api</artifactId>
        <scope>provided</scope>
    </dependency>

```

### concrete-core

cocnrete默认实现的核心逻辑模块，实现模块或者需要自行扩展concrete功能时依赖它。

### concrete-core-spring

concrete核心逻辑模块基于spring的一个实现，并且根据spring的特性提供了一些方便的组件。

### concrete-fsm, concrete-fsm-impl

一个轻量级的有限状态机模型

### concrete-test

concrete的单元调试模块

### concrete-rx

响应式编程的concrete支持模块

### cocnrete-support-jsr311

使用jaxrs 1.0发布服务时的依赖，不推荐使用

### concrete-support-jsr339

使用jaxrs 2.0发布服务时的依赖

###  cocnrete-support-websocket

使用websocket发布服务时的依赖

### cocnrete-support-dubbo

使用dubbo发布服务时的依赖

### concrete-api-tools

concrete api工具，用来生成文档、代码及脚本

### concrete-client

concrete服务的客户端调用模块

### concrete-jaxrs-client

使用jaxrs发布服务时，客户端的调用模块，需要额外依赖jaxrs的实现，推荐 `jersey-client`

### concrete-jaxrs-client-rx

使用jaxrs发布服务时，客户端的响应式调用模块，需要额外依赖jaxrs的实现，推荐 `jersey-client`

### concrete-dubbo-client concrete-dubboclient-rx

使用dubbo发布服务时，客户端的同步、响应式调用模块

### concrete-websocket-client-rx

使用websocket发布服务时，客户端的响应式调用模块

### concrete-commons-spring-data

spring-data的一些通用方法

### concrete-courier-jms

基于jms的发布订阅模型插件

### concrete-courier-jms-activemq

基于jms的发布订阅模型插件的activemq实现

### concrete-courier-rabbitmq

基于rabbitmq的发布订阅模型插件

### concrete-apm-zipkin

基于zipkin的应用性能管理插件

### concrete-apm-plugin-mysql

基于zipkin的应用性能管理的mysql插件
