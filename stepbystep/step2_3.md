# 2.3 依赖注入环境中使用Client

concrete-client也可以在依赖注入环境中使用。下面我们来走一个例子。

在`demo-client-invoker`的pom中，增加如下依赖

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core-spring</artifactId>
        </dependency>

        <dependency>
            <groupId>javax.inject</groupId>
            <artifactId>javax.inject</artifactId>
        </dependency>
```

> #### Hint::依赖包说明
>
> - concrete-core-spring: concrete基于spring的插件
> - javax.inject: jsr 330规范


新建 `org.coodex.concrete.demo.invoker.ClientInjectExample`

```java
package org.coodex.concrete.demo.invoker;

import org.coodex.concrete.ConcreteClient;
import org.coodex.concrete.demo.api.DemoService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import javax.inject.Inject;

public class ClientInjectExample {

    @Inject
    @ConcreteClient("demo")
    private DemoService demoService;

    public void example(){
        System.out.println(demoService.sayHello("inject"));
    }


    public static void main(String [] args){
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:example.xml");
        ClientInjectExample injectExample = context.getBean(ClientInjectExample.class);
        injectExample.example();
    }
}
```

简单明了，`@Inejct`声明这个属性需要被注入，`@ConcreteClient("demo")`声明这个属性使用client的`demo`模块配置

resources下增加一个`example.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config></context:annotation-config>
    <bean class="org.coodex.concrete.spring.ConcreteSpringConfiguration"/>
    <bean class="org.coodex.concrete.demo.invoker.ClientInjectExample"/>
</beans>
```

> #### Hint::说明
>
> - `context:annotation-config` 支持Annotation-based的依赖注入
> - `ConcreteSpringConfiguration`提供了concrete在spring实现中的一些关键Bean以及一些规范
> - `ClientInjectExample`,注册一下我们的例子

跑起来吧，看看控制台，别忘了把服务发布起来先。

与[2.2](step2_2.md)相比，这种方案更符合DI的思想。

截至到目前的代码： https://github.com/coodex2016/concrete-demo/tree/step2_3
