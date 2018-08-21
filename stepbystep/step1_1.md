# 1.1 开始定义api

api的pom中，增加concrete-api依赖
```xml
    <dependencies>
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-api</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

创建一个 interface `org.coodex.concrete.demo.api.DemoService`

> 关于包名，建议使用`**.模块.api` `××.模块.rpc` 用来存放Service定义，后续步骤中也会有相关说明

{% github_embed "https://github.com/coodex2016/concrete-demo/blob/step1/demo/api/src/main/java/org/coodex/concrete/demo/api/DemoService.java" %}{% endgithub_embed %}

以上，就是一个concrete service最基本的定义，几个要点：
- 必须是 `interface`
- 必须继承自 `org.coodex.concrete.api.ConcreteService`
- 被发布的服务接口必须有`@MicroService`注解
- 所有参数推荐加上 `@Parameter`注解，不一定和参数名一样，但是需要为该参数指定明确的名称

[1.2 实现api](step1_2.md)