# Concrete工具链

Concrete工具链提供了一套Concrete规范的参考实现。

## 从0开始

> 为了更加方便，concrete默认支持java8编译器的-parameters选项，请在ide中进行配置

### step 1

> 建立maven工程，导入concrete的pom

```xml

    <properties>
        <concrete.version>0.2.1-SNAPSHOT</concrete.version>
    </properties>


    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.coodex</groupId>
                <artifactId>concrete-bom</artifactId>
                <version>${concrete.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <compilerArgs>
                        <!-- 保留参数名，请使用java8及以上的jdk环境 -->
                        <arg>-parameters</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### step 2

> 建立api模块，依赖concrete-api

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-api</artifactId>
        </dependency>
```

> 定义API

```java
package org.coodex.demo.one;

import org.coodex.concrete.api.AccessAllow;
import org.coodex.concrete.api.ConcreteService;
import org.coodex.concrete.api.MicroService;

@MicroService("demo1") // 定义模块名，如果不定义，则使用concrete的默认规则
public interface Example extends ConcreteService /* API必须是一个ConcreteService */ {

    int add(int x1, int x2);

    @MicroService("vehicles")//定义资源名，详见restful风格
    /**
     * 针对jax-rs提供服务时，concrete有一套谓词管理，
     * 可以通过predicate.properties进行重载，
     * 详见 {@link https://book.concrete.coodex.org/%E5%B7%A5%E5%85%B7%E9%93%BE/jsr311.html#谓词定义 }
     */
    String getRandomVeh(String id);


    /**
     * rbac，声明该服务必须已登录用户才可访问
     *
     * 详见 {@link https://book.concrete.coodex.org/%E5%AE%9A%E4%B9%89/AccessAllow.html }
     * @return
     */
    @AccessAllow
    String aclTest();

}

```

### step 2.5， 跑起来看看

> 在api的test作用域【非必须】中依赖jeresy(jsr-339实现)、spring(jsr-330支持)、spring-boot(微服务、快速构建)、concrete的部分工具链

> 在api的test作用域这么干主要是为了偷懒，大家不要学我！大家不要学我！大家不要学我！

> 还有，这个过程只是为了演示一下，实际应用中切不可照抄照搬，一定要灵活运用！一定要灵活运用！一定要灵活运用！

```xml
        <!-- concrete基于spring的实现 -->
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core-spring</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- concrete 对jsr339的支持 -->
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-support-jsr339</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jersey</artifactId>
            <scope>test</scope>
        </dependency>
```

> spring boot的一个示例


```java
package org.coodex.demo.one;

import org.coodex.concrete.common.BeanProvider;
import org.coodex.concrete.core.intercept.ConcreteInterceptor;
import org.coodex.concrete.core.token.TokenManager;
import org.coodex.concrete.core.token.local.LocalTokenManager;
import org.coodex.concrete.jaxrs.ConcreteExceptionMapper;
import org.coodex.concrete.jaxrs.JaxRSServiceHelper;
import org.coodex.concrete.spring.SpringBeanProvider;
import org.coodex.concrete.spring.aspects.ConcreteAOPChain;
import org.coodex.concrete.spring.aspects.RBAC_Aspect;
import org.coodex.concrete.support.jsr339.javassist.JSR339ClassGenerator;
import org.glassfish.jersey.jackson.JacksonFeature;
import org.glassfish.jersey.logging.LoggingFeature;
import org.glassfish.jersey.server.ResourceConfig;
import org.glassfish.jersey.servlet.ServletContainer;
import org.glassfish.jersey.servlet.ServletProperties;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import java.util.ArrayList;
import java.util.List;

@SpringBootApplication //声明一个spring boot application，也就是说，这是一个spring boot框架定义中的微服务
@Configuration //懒得写xml而已
@EnableAspectJAutoProxy //同 <aop:aspectj-autoproxy/>
public class ServiceStart {

    @Bean
    public BeanProvider getBeanProvider() {
        // 同 <bean class="org.coodex.concrete.spring.SpringBeanProvider"/>
        return new SpringBeanProvider();
    }

    private List<ConcreteInterceptor> interceptors() {
        List<ConcreteInterceptor> interceptors = new ArrayList<>();
        interceptors.add(new RBAC_Aspect());//rbac 切片
        return interceptors;
    }

    @Bean
    public ConcreteAOPChain aspects() {
        // 同切片链的定义方式，参见 https://book.concrete.coodex.org/%E5%B7%A5%E5%85%B7%E9%93%BE/Aspects.html#切片链方式
        return new ConcreteAOPChain(interceptors());
    }

    @Bean
    public TokenManager getTokenManger() {
        // 同  <bean class="org.coodex.concrete.core.token.local.LocalTokenManager"/>
        return new LocalTokenManager();
    }


    public static class JerseyApplication extends ResourceConfig {
        public JerseyApplication() {
            registerClasses(JacksonFeature.class, // jersey自带的jackson json序列化
                    LoggingFeature.class,
                    ConcreteExceptionMapper.class);// concrete的异常mapper，统一异常处理

            registerClasses(
                    JaxRSServiceHelper.generate(
                            JSR339ClassGenerator.GENERATOR_NAME,
                            Example.class.getPackage().getName()));
        }
    }


    @Bean
    public ServletRegistrationBean testServlet() {
        ServletContainer container = new ServletContainer();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(
                container, "/service/*");
        registrationBean.addInitParameter(ServletProperties.JAXRS_APPLICATION_CLASS,
                JerseyApplication.class.getName());
        registrationBean.setName("demo");
        registrationBean.setAsyncSupported(true);
        return registrationBean;
    }


    public static void main(String [] args){
        // 设为前后端分离开发模式
        System.setProperty("org.coodex.concrete.jaxrs.devMode", "true");
        SpringApplication.run(ServiceStart.class, args);
    }

}

```

浏览器访问 http://localhost:8080/service/Demo1/vehicles/anyId 哒哒~~

ok，来看看客户端

#### java客户端

> 添加java客户端的依赖，concrete-jaxrs-client提供jaxrs的客户端支持，jersey-client提供jaxrs的client实现

```xml
        <!-- client example -->
        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-client</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs-client</artifactId>
            <scope>test</scope>
        </dependency>
```

> 客户端调用演示

```java
package org.coodex.demo.one.client;

import org.coodex.concrete.jaxrs.Client;
import org.coodex.demo.one.Example;

public class ClientExample {

    public static void main(String [] args){
        // 详见 https://book.concrete.coodex.org/%E5%B7%A5%E5%85%B7%E9%93%BE/JavaClient.html
        Example example = Client.getInstance(Example.class, "http://localhost:8080/service");

        System.out.println(example.getRandomVeh("id"));
    }
}
```

哒哒~~

#### jquery 客户端

增加api工具的依赖

```xml
        <!-- 这个是api工具 -->
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-api-tools</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- 为了省事，用spring boot web来跑前端 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <scope>test</scope>
        </dependency>
```

> 为了省事，演示过程使用spring boot web来跑前端，不是必须！不是必须！不是必须！

> 简单改造下

```java
public class ServiceStart extends SpringBootServletInitializer{ //继承自SpringBootServletInitializer
    ///....
}
```

生成api

```java
package org.coodex.demo.one.client;

import org.coodex.concrete.apitools.API;
import org.coodex.concrete.apitools.jaxrs.jquery.JQueryDocRender;
import org.coodex.concrete.apitools.jaxrs.jquery.JQueryPromisesCodeRender;
import org.coodex.demo.one.Example;

import java.io.IOException;

public class JQueryClientTool {

    public static void main(String [] args) throws IOException {
        // 详见 https://book.concrete.coodex.org/%E5%B7%A5%E5%85%B7%E9%93%BE/API.html
        API.generate(JQueryPromisesCodeRender.RENDER_NAME,
                "D:\\Projects\\IntelliJ\\concrete-demo\\one\\api\\src\\test\\resources\\static", //自己改，resource/static是spring-boot-web的默认静态资源目录
                Example.class.getPackage().getName());
    }
}
```

创建index.html

```html
<html>
<head>
    <script type="text/javascript" rel="script"
            src="https://cdnjs.cloudflare.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script src="jquery-concrete.js"></script>
</head>

<body>
<script>
    $(document).ready(function () {
        concrete.configure({
            root: '/service'
        });

        $('#demo').click(function () {
            concrete.module("Example").getRandomVeh("anyId").success(function (vehNum) {
                alert(vehNum);
            });
        });
    });
</script>

<button id="demo">vehicle(andyId)</button>
</body>
</html>
```

跑起来, http://localhost:8080 哒哒~~

> 演示是在同域下跑的，跨域时增加一个跨域filte就行

```java
    @Bean
    public FilterRegistrationBean corsFilter() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new CorsFilter());
        filterRegistrationBean.setAsyncSupported(true);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/service/*"));
        return filterRegistrationBean;
    }
```

angular/doc相关的，可以自己体验，需要说明的是，doc工具是基于gitbook的，请提前安装好

### step 3

实现接口，over 

:D

### step 4

to be continue, RxJava2 support in action ....
