# 1.2 实现api

建一个 `demo-impl` 子模块，模块目录为`demo/impl`

在pom中，引入以下依赖

```xml
    <dependencies>

        <dependency>
            <groupId>javax.inject</groupId>
            <artifactId>javax.inject</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>demo-api</artifactId>
            <version>${project.parent.version}</version>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core</artifactId>
        </dependency>

    </dependencies>
```

> 同工程的模块间，推荐使用项目变量的方式指定`groupId`和`version`

依赖`javax.inject`（proviced），这是因为，基于java的，我们应该遵循java ee规范，通过规范剥离对技术组件的依赖。concrete的出发点也是这样 

依赖`demo-api`，因为这个模块要实现它

依赖`concrete-core`，因为`concrete-core`提供了很多好用的东西，在这一步虽然用不到，先放着吧

> 关于依赖的顺序，考虑到maven依赖的就近原则，为了尽可能减少不通作用域的依赖冲突，建议按照`test` `scope` `complie` `system`的作用域顺序维护依赖

建一个实现类 `org.coodex.concrete.demo.impl.DemoServiceImpl`

> 关于包名，建议所有的实现且与实现相关的内容都放在 `**.模块.impl` 下，比如 `**.模块.impl.copier` `**.模块.impl.listener` `**.模块.impl.interceptor`等

```java
package org.coodex.concrete.demo.impl;

import org.coodex.concrete.demo.api.DemoService;

import javax.inject.Named;

@Named // 使用javax.inject规范，把该class定义成一个可被注入的对象
public class DemoServiceImpl implements DemoService {
    @Override
    public int add(int x1, int x2) {
        return x1 + x2;
    }

    @Override
    public String sayHello(String name) {
        return String.format("Hello %s!", name);
    }
}
```

[1.3 使用Spring boot 跑起来](step1_3.md)