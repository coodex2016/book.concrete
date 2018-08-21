# 开始定义api

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

```java
package org.coodex.concrete.demo.api;

import org.coodex.concrete.api.ConcreteService;
import org.coodex.concrete.api.MicroService;
import org.coodex.util.Parameter;

@MicroService()
public interface DemoService extends ConcreteService {

    /**
     * x1 + x2 = ?
     *
     * @param x1
     * @param x2
     * @return x1 + x2
     */
    int add(@Parameter("x1") int x1,
            @Parameter("x2") int x2);

    /**
     * say hello to %name%
     *
     * @param name
     * @return hello %name%
     */
    String sayHello(@Parameter("name") String name);

}
```

以上，就是一个concrete service最基本的定义，几个要点：
- 必须是 `interface`
- 必须继承自 `org.coodex.concrete.api.ConcreteService`
- 所有参数推荐加上 `@Parameter`注解，不一定和参数名一样，但是需要为该参数指定明确的名称

[02 实现api](step2.md)