# 1.3 使用spring boot 跑起来

建一个`demo-release`子模块，目录位置`demo/release`

在模块的pom中增加如下依赖

```xml
    <dependencies>

        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>demo-impl</artifactId>
            <version>${project.parent.version}</version>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core-spring</artifactId>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-support-jsr339</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jersey</artifactId>
        </dependency>

    </dependencies>
```

> concrete-core-spring: concrete基于spring的插件  
> concrete-support-jsr339: jaxrs2.0规范的服务端支持组件  
> spring-boot-starter-jersey: spring boot的jersey插件，jersey是oracle官方的一个jaxrs参考实现

新建一个class `org.coodex.concrete.demo.boot.DemoBoot`

```java
package org.coodex.concrete.demo.boot;

import org.coodex.concrete.demo.api.DemoService;
import org.coodex.concrete.spring.ConcreteSpringConfiguration;
import org.coodex.concrete.support.jsr339.ConcreteJSR339Application;
import org.glassfish.jersey.jackson.JacksonFeature;
import org.glassfish.jersey.servlet.ServletContainer;
import org.glassfish.jersey.servlet.ServletProperties;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@SpringBootApplication(scanBasePackages = "org.coodex.concrete.demo.impl")
@Configuration
@Import(ConcreteSpringConfiguration.class)
public class DemoBoot {

    /**
     * 运行 spring boot
     *
     * @param args
     */
    public static void main(String[] args) {
        SpringApplication.run(DemoBoot.class, args);
    }

    /**
     * 定义一个jaxrsServlet
     *
     * @return
     */
    @Bean
    public ServletRegistrationBean jaxrsServlet() {
        ServletContainer container = new ServletContainer();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(
                container, "/jaxrs/*");

        // 按照jaxrs规范，指定Application的className
        registrationBean.addInitParameter(ServletProperties.JAXRS_APPLICATION_CLASS,
                JaxRSApplication.class.getName());

        registrationBean.setName("demo");

        // 异步支持（重要：servlet 3.0/jaxrs 2.0 都支持异步，性能和可管理性都有大幅提升）
        registrationBean.setAsyncSupported(true);
        return registrationBean;
    }

    /**
     * jsr339(jaxrs 2.0)规范的Application
     */
    public static class JaxRSApplication extends ConcreteJSR339Application {
        public JaxRSApplication() {
            register(
                    // 使用 jackson 作为 jaxrs的序列化和反序列化实现
                    JacksonFeature.class,
                    // 注册 我们写的api
                    DemoService.class);
        }
    }
}

```

> scanBasePackages 指定 `org.coodex.concrete.demo.impl`，这也是为什么我们要这么命名的原因，`“约定优于配置”`，我们的项目应该有一个统一的命名约定  
> `@Import(ConcreteSpringConfiguration.class)`, `ConcreteSpringConfiguration`中预制了很多concrete所需的bean和约定


run

访问以下地址：

http://localhost:8080/jaxrs/OrgCoodexConcreteDemoApiDemoService/add/1/2

http://localhost:8080/jaxrs/OrgCoodexConcreteDemoApiDemoService/sayHello/davidoff

> 针对上述链接，我们做一下解析：  
> http://localhost:8080  spring boot 默认的主机和端口  
> /jaxrs <-- 我们注册的jaxrsServlet的mapping  
> /OrgCoodexConcreteDemoApiDemoService <- concrete jaxrs 默认的服务类命名方式，去掉`.`，转换为大驼峰。好麻烦是吗？没关系，一会我们看看怎么让它更方便  
> /add <-- 方法名  
> /1/2 <-- 参数x1和x2  
>   
> 实际上，我们并不需要关注这些，[concrete-api-tools](../impl/API.md)能够帮我们生成与访问有关的信息，
> 开发者无需关注类、方法、参数相关的形式，直接按照规范调用就好了。当我们需要对外部非concrete调用时，
> 也可以通过[concrete-api-tools](../impl/API.md)生成服务手册提供给第三方，为了让文档更加丰富，
> 我们将使用到concrete的[文档化](../definition/Description.md)规范


通过这个过程，我们可以对比一下concrete与jersey或spring mvc在开发方式上的差异，单纯作为服务端来讲，
可能工作量上不相上下，但是，concrete单独定义了interface，为服务的调用者提供了更方便的使用方式，而且，
jaxrs只是concrete支持的方式之一，目前还有`dubbo` `websocket`的支持，后续还会提供更多的服务发布模式支持

截止到目前，以上的源代码保留在 https://github.com/coodex2016/cocnrete-demo/tree/step1

好了，强调一个概念，系统与系统之间，模块与模块之间都是通过`接口`来进行耦合的，那么系统与系统之间，模块与模块之间如何做到协同呢？ 
concrete给出的答案是：  
- 系统与系统之间，非java环境或非java6以上环境，提供[服务文档](step2_1.md)
- 系统与系统之间，java6以上环境，使用[`concrete-client`](step2_2.md)
- 模块与模块之间，使用`concrete-client`的[模块化](step2_3.md)
- Browser前端到系统之间，使用`concrete-api-tools`[生成代码](step2_4.md)
