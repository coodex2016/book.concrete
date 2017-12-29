# Concrete工具链

Concrete工具链提供了一套Concrete规范的参考实现。

## 从0开始

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
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### step 2

> 建立api模块（示例的artifactId为example-api），依赖concrete-api

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-api</artifactId>
        </dependency>
```

> 定义API

```java
package org.coodex.example;

import org.coodex.concrete.api.AccessAllow;
import org.coodex.concrete.api.ConcreteService;
import org.coodex.concrete.api.Description;
import org.coodex.concrete.api.MicroService;

@MicroService("example")
public interface ExampleApi extends ConcreteService {

    @Description(name = "求和")
    int add(
            @Parameter("x1") @Description(name = "被加数") int x1,
            @Parameter("x2") @Description(name = "加数") int x2);

    @MicroService("vehicles")//定义资源名，详见restful风格
    /**
     * 针对jax-rs提供服务时，concrete有一套谓词管理，
     * 可以通过predicate.properties进行重载，
     * 详见 {@link https://concrete.coodex.org/%E5%B7%A5%E5%85%B7%E9%93%BE/jsr311.html#谓词定义 }
     */
    String getRandomVeh(
            @Parameter("id") String id);


    /**
     * rbac，声明该服务必须已登录用户才可访问
     * <p>
     * 详见 {@link https://concrete.coodex.org/%E5%AE%9A%E4%B9%89/AccessAllow.html }
     */
    @AccessAllow
    String aclTest();

}

```

### step 2.5， 跑起来，简单体验一下工具链

> 建一个release-jaxrs模块
> 增加jeresy(jsr-339实现)、spring(jsr-330支持)、spring-boot(微服务、快速构建)、concrete的部分工具链的依赖
> 重要的，这个过程只是为了演示一下，实际应用中切不可照抄照搬，一定要灵活运用！一定要灵活运用！一定要灵活运用！

```xml

        <!-- 依赖api -->
        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>example-api</artifactId>
            <version>${project.parent.version}</version>
        </dependency>


        <!-- concrete基于spring的实现 -->
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core-spring</artifactId>
        </dependency>

        <!-- concrete 对jsr339的支持 -->
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-support-jsr339</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jersey</artifactId>
        </dependency>
```

> spring boot的一个示例


```java
package org.coodex.example;

import org.coodex.concrete.core.intercept.ConcreteInterceptor;
import org.coodex.concrete.core.intercept.RBACInterceptor;
import org.coodex.concrete.core.token.TokenManager;
import org.coodex.concrete.core.token.local.LocalTokenManager;
import org.coodex.concrete.spring.ConcreteSpringConfiguration;
import org.coodex.concrete.spring.aspects.ConcreteAOPChain;
import org.coodex.concrete.support.jsr339.ConcreteJaxrs339Application;
import org.glassfish.jersey.jackson.JacksonFeature;
import org.glassfish.jersey.logging.LoggingFeature;
import org.glassfish.jersey.servlet.ServletContainer;
import org.glassfish.jersey.servlet.ServletProperties;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.context.annotation.Import;

import java.util.ArrayList;
import java.util.List;

@SpringBootApplication //声明一个spring boot application，也就是说，这是一个spring boot框架定义中的微服务
@Configuration
@EnableAspectJAutoProxy //同 <aop:aspectj-autoproxy/>
@Import(ConcreteSpringConfiguration.class)// 同<bean class="org.coodex.concrete.spring.ConcreteSpringConfiguration"/>
public class SpringBootWithoutXML {

    private List<ConcreteInterceptor> interceptors() {
        List<ConcreteInterceptor> interceptors = new ArrayList<>();
        interceptors.add(new RBACInterceptor());//rbac 切片
        return interceptors;
    }


    // 同切片链的定义方式，参见 https://book.concrete.coodex.org/%E5%B7%A5%E5%85%B7%E9%93%BE/Aspects.html#切片链方式
    @Bean
    public ConcreteAOPChain aspects() {
        return new ConcreteAOPChain(interceptors());
    }


    // 同  <bean class="org.coodex.concrete.core.token.local.LocalTokenManager"/>
    @Bean
    public TokenManager getTokenManger() {
        return new LocalTokenManager();
    }

    /**
     * jaxrs规范的Application
     */
    public static class ExampleApplication extends ConcreteJaxrs339Application{

        public ExampleApplication() {
            register(JacksonFeature.class,// 使用jersey的jackson插件序列化
                    LoggingFeature.class,
                    ExampleApi.class);// 注册服务Api，需要发布哪些服务就注册哪些，也可以使用registerPackage按包注册

        }
    }

    @Bean
    public ServletRegistrationBean testServlet() {
        ServletContainer container = new ServletContainer();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(
                container, "/jaxrs/*");
        registrationBean.addInitParameter(ServletProperties.JAXRS_APPLICATION_CLASS,
                ExampleApplication.class.getName());
        registrationBean.setName("demo");
        registrationBean.setAsyncSupported(true);
        return registrationBean;
    }


    public static void main(String [] args){
        // 设为前后端分离开发模式
        System.setProperty("org.coodex.concrete.jaxrs.devMode", "true");
        SpringApplication.run(SpringBootWithoutXML.class, args);
    }

}


```

浏览器访问 http://localhost:8080/jaxrs/Example/vehicles/anyId 哒哒~~

好啦，现在看到的是一串随机的字符串，玩个小游戏，让它返回的数据更贴近前端开发的需要。

```java
    @VehicleNum // <--增加个这个
    String getRandomVeh(String id);
```

跑起来，哒哒~~~

#### 关于apiTools

concrete的理念是把详细设计的文档和设计代码工作合并起来，通过一份工作成果来产生与之一致化的多份工作产品（api文档、rxSDK、jQuery based SDK、Angular based SDK）

##### api文档

我们在release模块中来生成文档

> 不是必须的，需要根据实际的项目结构和你想文档化的api范围来确定

在release模块增加api-tools的依赖
```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-api-tools</artifactId>
            <scope>test</scope>
        </dependency>
```

新建APIGenerator类

```java
package org.coodex.example;

import org.coodex.concrete.apitools.API;
import org.coodex.concrete.apitools.jaxrs.service.ServiceDocRender;

import java.io.IOException;

public class APIGenerator {

    public static void main(String [] args) throws IOException {
        API.generate(ServiceDocRender.RENDER_NAME, "/doc/restful",
                ExampleApi.class.getPackage().getName());
    }
}
```

run, `cd /doc/restful`, `gitbook install` (安装插件，只需要执行一次), `gitbook serve`
cocnrete文档化是基于gitbook的，所以请先安装好`gitbook-cli`

浏览器访问 http://localhost:4000 哒哒

### step 3, 实现接口

根据定义来进行实现的编码，这是程序员们最擅长的工作，不过在这里，我们将在这份体验代码中体验更多的jsr。
并且尽可能让这份代码能够成为一个模板，有些通用的内容，大家复制一下就行了。

> 建立impl模块，依赖api、concrete基于spring的工具链及有关jsr。
> 简单说一下，spring在concrete中，我们更多的是用到它作为jsr330（依赖注入）的实现，不过一会的实现中，我们会用到spring更多的效率工具

```xml
        <!-- 依赖api -->
        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>example-api</artifactId>
            <version>${project.parent.version}</version>
        </dependency>


        <!-- concrete基于spring的实现 -->
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core-spring</artifactId>
        </dependency>
        
        <!-- 笼统的把javaee7的api引入进来,javaee api实质上就是部分jsr的合集 -->
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
```

> 编写一个ExampleApi的实现

```java
package org.coodex.example;

public class ExampleImpl implements ExampleApi {
    @Override
    public int add(int x1, int x2) {
        return x1 + x2;
    }

    @Override
    public String getRandomVeh(String id) {
        return "不要打了，我就是车牌，行了吧？";
    }

    @Override
    public String aclTest() {
        return null;
    }
}
```

> 把之前release的依赖改成依赖impl

```xml
        <!-- 依赖 impl -->
        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>example-impl</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
```

> 在SpringBootWithout注册一个ExampleApi类型的bean，并去掉那行前后端分离开发的设置

```java
    @Bean
    public ExampleApi getExampleApi(){
        return new ExampleImpl();
    }
```

run起来，分别看看 
> http://localhost:8080/jaxrs/Example/add/1/1
> http://localhost:8080/jaxrs/Example/vehicles/anyId
> http://localhost:8080/jaxrs/Example/aclTest

ok,看到了吗，aclTest因为没有登录，所以会报1005错误。这个错误码是concrete定义体系中的一部分，详见[ErrorCodes](../定义/AbstractErrorCodes.md).

目前这个响应并不太友好，我们基于concrete默认的错误信息加载模式把错误信息配置补上

> 在api模块中，建两个properties，一个用于concrete的core errorCodes，一个等会用，命名，一个叫concreteCore.properties,一个叫example.properties

```properties
## concreteCore.properties
## 我们改掉1005
message.0=OK
message.1001=service definition not found at {0}, {1}.
message.1002=method definition not found in [{0}]: {1}.
message.1003=none token.
message.1004=token invalidate: {0}.
message.1005=用户尚未登录
message.1006=NO AUTHORIZATION.
message.1007=ACCOUNT INVALIDATE.
message.1008=UNTRUSTED ACCOUNT.
message.1009=violation: {0}.
message.1010=NO BEAN PROVIDER FOUND.
message.1011=<NO SERVICE INSTANCE FOUND> {0}.
message.1012=BEAN CONFLICT {0}, {1}.
message.1013=OUT OF SERVICE TIME.
message.1014=OVERRUN.
message.1015=SIGNING FAILED: {0}.
message.1016=SIGNATURE VERIFICATION FAILED: {0}.
message.1017=Unknown class: {0}
message.1018=MODULE DEFINITION NON UNIQUENESS: {0}
message.99998=Client Error.
message.99999=|UNKNOWN ERROR|: {0}.
```

> 然后在release模块，增加concrete.properties，并在其中指定所使用的错误代码信息模板

```properties
# concrete.properties

messagePattern.resourceBundles=concreteCore, example
```

重启一下，再看看aclTest，哒哒

concrete工具链会根据客户端的语言环境查找合适的资源文件，因此，针对错误信息，我们可以方便的按照java resource bundle的方式达到I18n

好啦，继续，玩点花花的，体验一些jsr和spring的效率工具

我们重新设计一下求和以及获取车牌的实现。我们利用JPA（jsr规范）和spring-data-jpa(jpa的效率工具)来实现。

> impl模块中，增加spring-data-jpa的依赖

```xml
        <!-- spring data jpa -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
        </dependency>
```

> 定义复合id类型AddId，实体AddEntity

```java
package org.coodex.example.entities;

import java.io.Serializable;
import java.util.Objects;

public class AddId implements Serializable {
    private int x1;
    private int x2;

    public AddId() {
    }

    public AddId(int x1, int x2) {
        this.x1 = x1;
        this.x2 = x2;
    }

    public int getX1() {
        return x1;
    }

    public void setX1(int x1) {
        this.x1 = x1;
    }

    public int getX2() {
        return x2;
    }

    public void setX2(int x2) {
        this.x2 = x2;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        AddId addId = (AddId) o;
        return x1 == addId.x1 &&
                x2 == addId.x2;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x1, x2);
    }
}
```

```java
package org.coodex.example.entities;

import javax.persistence.*;

@Entity
@Table(name = "t_add")
@IdClass(AddId.class)
public class AddEntity {
    @Id
    private int x1;
    @Id
    private int x2;

    @Column(name = "col_sum")
    private int sum;

    public int getX1() {
        return x1;
    }

    public void setX1(int x1) {
        this.x1 = x1;
    }

    public int getX2() {
        return x2;
    }

    public void setX2(int x2) {
        this.x2 = x2;
    }

    public int getSum() {
        return sum;
    }

    public void setSum(int sum) {
        this.sum = sum;
    }
}
```

> 使用神奇的spring-data-jpa定义DAO接口

```java
package org.coodex.example.repos;

import org.coodex.example.entities.AddEntity;
import org.coodex.example.entities.AddId;
import org.springframework.data.repository.CrudRepository;

public interface AddEntityRepo extends CrudRepository<AddEntity, AddId> {
}

```

好啦，我们开始改造实现。逻辑为：到数据库里看有没有x1+x2的记录，如果有，返回sum，否则产生一个“太难了”的异常

> 在api模块中定义ExampleErrorCodes及其错误信息模板

```java
package org.coodex.example;

import org.coodex.concrete.common.AbstractErrorCodes;

public class ExampleErrorCodes extends AbstractErrorCodes {
    protected final static int BASE = CUSTOM_LOWER_BOUND + 5000;

    public static final int TOO_HARD = BASE + 1;
}
```

```properties
# example.properties
message.105001={0} + {1}太难了，我不会
```

> 在ExampleImpl中注入DAO对象

```java
    @Inject
    private AddEntityRepo addEntityRepo;
```

> 修改add接口实现

```java
    public int add(int x1, int x2) {
        return Assert.isNull(
                addEntityRepo.findOne(new AddId(x1, x2)),
                ExampleErrorCodes.TOO_HARD, x1, x2).getSum();
    }
```

代码搞定，下面开始准备数据和spring配置

> 准备数据，我们往数据库里加0到9的加和数据，同样的，使用spring-data-jpa
> 在impl模块的test作用域里，依赖jpa实现和数据库连接的相关内容

```xml
        <!-- jpa的实现 hibernate-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>test</scope>
        </dependency>
```

> spring-data-jpa配置 test/resources/spring-data-example.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url"
                  value="jdbc:mysql://localhost:3306/concrete?useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="concrete-user"/>
        <property name="password" value="pwd"/>
    </bean>

    <!-- JPA Vendor - Hibernate -->
    <bean id="hibernateJpaVendorAdapter"
          class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>

    <!-- entity manager -->
    <bean id="entityManagerFactory"
          class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">

        <property name="dataSource" ref="dataSource"/>

        <!-- use hibernate -->
        <property name="jpaVendorAdapter" ref="hibernateJpaVendorAdapter"/>
        <!-- 指定Entity实体类包路径 -->
        <property name="packagesToScan">
            <array>
                <value>org.coodex.example.entities</value>
            </array>
        </property>

        <!-- JPA Vendor Settings : for hibernate -->
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <!-- hibernate cache, 根据系统需要进行选择,暂定不需要cache -->
                <prop key="hibernate.cache.provider_class">org.hibernate.cache.NoCacheProvider</prop>
                <!-- debug -->
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>
                <!-- auto update ddl -->
                <prop key="hibernate.hbm2ddl.auto">update</prop>
            </props>
        </property>
    </bean>

    <!-- transaction manager -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory" />
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager" />

    <jpa:repositories
            entity-manager-factory-ref="entityManagerFactory"
            transaction-manager-ref="transactionManager"
            base-package="org.coodex.example.repos"/>


</beans>
```

> 编写增加数据的逻辑

```java
package org.coodex.example;

import org.coodex.example.entities.AddEntity;
import org.coodex.example.repos.AddEntityRepo;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class DataPrepared {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("spring-data-example.xml");
        AddEntityRepo addEntityRepo = context.getBean(AddEntityRepo.class);
        for (int x1 = 0; x1 < 10; x1++) {
            for (int x2 = 0; x2 < 10; x2++) {
                AddEntity addEntity = new AddEntity();
                addEntity.setX1(x1);
                addEntity.setX2(x2);
                addEntity.setSum(x1 + x2);
                addEntityRepo.save(addEntity);
            }
        }
    }
}
```

run it. ok, 数据库里已经有0+0 到9+9的数据了

这个时候，问题来了。我们的实现模块其实并不关JPA的实现和数据库的链接，而release模块虽然关注这些，但是它更关注的是如何提供服务，比如是jaxrs还是websocket，而这些又是跟选择什么样的数据库、什么样的jpa实现无关的，怎么办呢？

我们新建一个配置模块来隔离impl和release对jpa相关的耦合，并约定使用spring-data-env.xml来配置jpa环境

> 建立example-configuration-hibernate-mysql模块，使用hibernate+mysql+druid来构建jpa的数据环境

```xml
        <!-- jpa的实现 hibernate-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>

        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>example-impl</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
```

> spring-data-env.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${org.coodex.example|ds.driverClassName:com.mysql.jdbc.Driver}"/>
        <property name="url"
                  value="${org.coodex.example|ds.url:jdbc:mysql://localhost:3306/concrete?useUnicode=true&amp;characterEncoding=UTF-8}"/>
        <property name="username" value="${org.coodex.example|ds.username:concrete-user}"/>
        <property name="password" value="${org.coodex.example|ds.pwd:pwd}"/>
    </bean>

    <!-- JPA Vendor - Hibernate -->
    <bean id="hibernateJpaVendorAdapter"
          class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>

    <!-- entity manager -->
    <bean id="entityManagerFactory"
          class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">

        <property name="dataSource" ref="dataSource"/>

        <!-- use hibernate -->
        <property name="jpaVendorAdapter" ref="hibernateJpaVendorAdapter"/>
        <!-- 指定Entity实体类包路径 -->
        <property name="packagesToScan">
            <array>
                <value>org.coodex.example.entities</value>
            </array>
        </property>

        <!-- JPA Vendor Settings : for hibernate -->
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.dialect">${org.coodex.example|hibernate.dialect:org.hibernate.dialect.MySQL5Dialect}</prop>
                <prop key="hibernate.cache.provider_class">${org.coodex.example|hibernate.cache.provider_class:org.hibernate.cache.NoCacheProvider}</prop>
                <prop key="hibernate.show_sql">${org.coodex.example|hibernate.show_sql:false}</prop>
                <prop key="hibernate.format_sql">${org.coodex.example|hibernate.format_sql:true}</prop>
                <prop key="hibernate.hbm2ddl.auto">${org.coodex.example|hibernate.hbm2ddl.auto:update}</prop>
            </props>
        </property>
    </bean>

    <!-- transaction manager -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory" />
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager" />

    <jpa:repositories
            entity-manager-factory-ref="entityManagerFactory"
            transaction-manager-ref="transactionManager"
            base-package="org.coodex.example.repos"/>


</beans>
```

> 修改release的依赖

```xml
        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>example-configuration-hibernate-mysql</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
```

> 引入xml
```java
@ImportResource("classpath:spring-data-env.xml")
```
run. 然后分别 http://localhost:8080/jaxrs/Example/add/1/4 和 http://localhost:8080/jaxrs/Example/add/10/4 


### step 4, 前端sdk

concrete服务端目前提供了jaxrs和websocket的服务支持，同样的前端sdk也提供了针对jaxrs和websocket的支持，分别基于jQuery和angular。JavaClient也提供了对这两者的支持，分别基于同步调用和rx

#### jaxrs

##### jquery

为了方便演示，我们使用spring-boot-start-web来发布jquery前端静态资源。

> release模块增加spring-boot-start-web依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

> 改造SpringBootWithoutXML，继承`SpringBootServletInitializer`，ok，默认`resources/static`就是静态资源的根，当然，你可以通过Spring的 profile: application.properties去改

> 在release模块的test域中生成jQuery-jaxrs的sdk到'resources/static/jquery'

```java
package org.coodex.example;

import org.coodex.concrete.apitools.API;
import org.coodex.concrete.apitools.jaxrs.jquery.JQueryPromisesCodeRender;

import java.io.IOException;

public class JQuerySDKGenerator {

    public static void main(String [] args) throws IOException {
        String projectPath = "path/to/your/project/root";// 自行修改
        API.generate(JQueryPromisesCodeRender.RENDER_NAME,
                projectPath + "/src/main/resources/static/jquery",
                ExampleApi.class.getPackage().getName());
    }
}

```

这样，我们在`resources/static/jquery`下可以看到一个`jquery-cocnrete.js`，它就是根据我们api模块定义生成的sdk。

下面做个简单的前端页面保存为'在`resources/static/jquery/index.html`

```html
<!DOCTYPE>
<html>
<head>
    <title>jQuery SDK example</title>
    <script type="text/javascript" rel="script"
            src="https://cdnjs.cloudflare.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script src="jquery-concrete.js"></script>
    <script>
        $(document).ready(function () {
            // 设置服务根
            concrete.configure({
                root: '/jaxrs'
            });

            var exampleModule = concrete.module("ExampleApi");

            $('#add45').click(function () {
                exampleModule.add(4,5).then(function (sum) {
                    alert("4 + 5 = " + sum);
                })
            });

            $('#add145').click(function () {
                exampleModule.add(14,5).then(function (sum) {
                    alert("14 + 5 = " + sum);
                })
            });
        });
    </script>
</head>
<body>
    <button id="add45">add(4, 5)</button>
    <button id="add145">add(14, 5)</button>
</body>
</html>
```

跑起来，http://localhost:8080/jquery/index.html 哒哒

##### angular

> 先建立angular工程

```
ng new angular-jaxrs
```

> 修改tsconfig.json，增加"baseUrl"属性，用于搜索sdk

```json
{
    "compileOnSave": false,
    "compilerOptions": {
        "baseUrl": "src",
        "outDir": "./dist/out-tsc",
        "sourceMap": true,
        "declaration": false,
        "moduleResolution": "node",
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "target": "es5",
        "typeRoots": [
            "node_modules/@types"
        ],
        "lib": [
            "es2017",
            "dom"
        ]
    }
}
```

> 创建Angular SDK

```java
package org.coodex.example;

import org.coodex.concrete.apitools.API;
import org.coodex.concrete.apitools.jaxrs.angular.AngularCodeRender;

import java.io.IOException;

public class AngularSDKGenerator {

    public static void main(String [] args) throws IOException {

        String angularProject = "path/to/your/angular/project";
        API.generate(AngularCodeRender.RENDER_NAME,
                angularProject + "/src",
                ExampleApi.class.getPackage().getName());
    }
}

```
这时候可以看看src目录，下面创建了@concrete，这就是针对ExampleApi的SDK了，我们修改一下访问根（@concrete/AbstractConcreteService.ts）

```typescript
// ....
    public root: string = 'http://localhost:8080/jaxrs';
// ....
```

> 增加concrete sdk的调用代码

```
ng g c example
```

```typescript
import {BrowserModule} from '@angular/platform-browser';
import {NgModule} from '@angular/core';
import {FormsModule} from '@angular/forms'; // <-- here


import {AppComponent} from './app.component';
import {ExampleApi} from "@concrete/org/coodex/example"; // <-- here
import { ExampleComponent } from './example/example.component';


@NgModule({
    declarations: [
        AppComponent,
        ExampleComponent
    ],
    imports: [
        BrowserModule,
        FormsModule  // <-- here
    ],
    providers: [ExampleApi], // <-- here
    bootstrap: [AppComponent]
})
export class AppModule {
}

```

> 修改example模板

```html
<div>
    x1: <input type="number" [(ngModel)]="x1"/>
    x2: <input type="number" [(ngModel)]="x2"/>
    <button (click)="add()">add</button>
</div>

```

> example Component

```typescript
import {Component, OnInit} from '@angular/core';
import {ExampleApi} from "@concrete/org/coodex/example";

@Component({
    selector: 'app-example',
    templateUrl: './example.component.html',
    styleUrls: ['./example.component.css']
})
export class ExampleComponent implements OnInit {

    private x1: number;
    private x2: number;

    constructor(private example: ExampleApi) {
    }

    ngOnInit() {
        this.x1 = 0;
        this.x2 = 0;
    }
    
    add(){
        this.example.add(this.x1,this.x2).subscribe(
            value => alert(this.x1 + " + " + this.x2 + " = " + value),
            (error) => alert(error));
    }
}

```

把 app-example组件添加到app component中，然后 ng serve

这时候我们可以看到，前端是跑在http://localhost:4200 而服务端是http://localhost:8080，点击add时会发起一个option请求
但是server端并没有给出响应，为了支持跨域，我们在server端增加corsFilter及相关的配置

> release模块增加coodex-utilities-servlet依赖

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>coodex-utilities-servlet</artifactId>
        </dependency>
```

> 修改 SpringBootWithoutXML

```java
    @Bean
    public FilterRegistrationBean corsFilter() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new CorsFilter());// 注意，是coodex utilites的CorsFilter
        filterRegistrationBean.setAsyncSupported(true);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/jaxrs/*"));
        return filterRegistrationBean;
    }
```

> cors_settings.properties，注意，allowOrigin用宽泛的定义不适合生产环境

```properties
allowOrigin= *
exposeHeaders= concrete-token-id,concrete-error-occurred,content-type
allowMethod= POST,OPTIONS,GET,DELETE,PUT,PATCH
allowHeaders= CONCRETE-TOKEN-ID,CONTENT-TYPE,X-CLIENT-PROVIDER
allowCredentials= true
```

好了，跑起来看看，哒哒~

好吧，写文档的时候，发现HttpModule已经被deprecated了，再看看angular版本号，5.1.2，更新太快了。文档完成后再仔细关注一下angular版本更新信息，然后根据不同的angular版本提供sdk

##### java 客户端sdk

java 客户端 sdk是为java客户端环境准备的（Android、服务端互相调用），毕竟是一套体系，应用起来也相对简单。特别提一点，服务端模块间互访尽量使用JavaClient来请求，这样在系统架构上会更灵活，方便的兼容部署在同一个虚拟机和不同虚拟机模式。

###### 同步调用模式

这个最简单了，我们在release的test作用域里增加java client相关的依赖

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs-client</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-client</artifactId>
            <scope>test</scope>
        </dependency>
```

> test作用域里写个体验类

```java
package org.coodex.example;

import org.coodex.concrete.jaxrs.Client;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class JavaClientTest {

    private final static Logger log = LoggerFactory.getLogger(JavaClientTest.class);


    public static void main(String [] args){
        ExampleApi example = Client.getInstance(ExampleApi.class, "http://localhost:8080/jaxrs");
        log.debug("{} + {} = {}", 1, 2, example.add(1,2));
        try{
            log.debug("{} + {} = {}", 13, 2, example.add(13,2));
        }catch (Throwable th){
            log.error(th.getLocalizedMessage(), th);
        }
    }
}
```


###### 响应式编程模式

> 增加响应式编程模式依赖

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs-client-rx</artifactId>
            <scope>test</scope>
        </dependency>
```

> 生成响应式接口

```java
package org.coodex.example;

import org.coodex.concrete.apitools.API;
import org.coodex.concrete.apitools.rx.ReactiveStreamsRender;

import java.io.IOException;

public class RXGenerator {

    public static void main(String [] args) throws IOException {
        String projectPath = "path/to/your/project";
        API.generate(ReactiveStreamsRender.RENDER_NAME,
                projectPath + "/src/test/java",
                ExampleApi.class.getPackage().getName());
    }
}

```

这样会生成指定的rx接口，在本例中，在test scope中生成了`rx.org.coodex.example.ExampleApi_RX`

> 调用

```java
package org.coodex.example;

import io.reactivex.Observer;
import io.reactivex.disposables.Disposable;
import org.coodex.concrete.rx.RXClient;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import rx.org.coodex.example.ExampleApi_RX;

public class RxTest {

    private final static Logger log = LoggerFactory.getLogger(RxTest.class);

    public static void main(String[] args) {
        ExampleApi_RX example = RXClient.getInstance(ExampleApi_RX.class, "http://localhost:8080/jaxrs");
        example.add(1, 2).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(Integer integer) {
                log.debug("1 + 2 = {}", integer);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
    }
}
```

大概齐就体验这些吧，哦，对了，websocket支持

### step 5 另一种服务协议 jsr356 webSocket

使用上相差不大，快速过一遍

> 建立release-websocket模块，添加依赖

```xml
        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>example-configuration-hibernate-mysql</artifactId>
            <version>${project.parent.version}</version>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-support-websocket</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jersey</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- 默认的json序列化支持-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
        </dependency>
```

> 一样的，建个booter，不同的是，需要定义ConcreteWebSocketEndPoint类

```java
package org.coodex.example;

import org.coodex.concrete.core.intercept.ConcreteInterceptor;
import org.coodex.concrete.core.intercept.RBACInterceptor;
import org.coodex.concrete.core.token.TokenManager;
import org.coodex.concrete.core.token.local.LocalTokenManager;
import org.coodex.concrete.spring.ConcreteSpringConfiguration;
import org.coodex.concrete.spring.aspects.ConcreteAOPChain;
import org.coodex.concrete.support.websocket.CallerHackConfigurator;
import org.coodex.concrete.support.websocket.ConcreteWebSocketEndPoint;
import org.glassfish.jersey.servlet.ServletContainer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.boot.web.support.SpringBootServletInitializer;
import org.springframework.context.annotation.*;

import javax.servlet.ServletContext;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.ServletException;
import javax.websocket.DeploymentException;
import javax.websocket.server.ServerContainer;
import javax.websocket.server.ServerEndpoint;
import java.util.ArrayList;
import java.util.List;

@SpringBootApplication //声明一个spring boot application，也就是说，这是一个spring boot框架定义中的微服务
@Configuration
@EnableAspectJAutoProxy //同 <aop:aspectj-autoproxy/>
@Import(ConcreteSpringConfiguration.class)// 同<bean class="org.coodex.concrete.spring.ConcreteSpringConfiguration"/>
@ImportResource("classpath:spring-data-env.xml")
public class SpringBootWithoutXML
        extends SpringBootServletInitializer {

    private List<ConcreteInterceptor> interceptors() {
        List<ConcreteInterceptor> interceptors = new ArrayList<>();
        interceptors.add(new RBACInterceptor());//rbac 切片
        return interceptors;
    }


    // 同切片链的定义方式，参见 https://book.concrete.coodex.org/%E5%B7%A5%E5%85%B7%E9%93%BE/Aspects.html#切片链方式
    @Bean
    public ConcreteAOPChain aspects() {
        return new ConcreteAOPChain(interceptors());
    }


    // 同  <bean class="org.coodex.concrete.core.token.local.LocalTokenManager"/>
    @Bean
    public TokenManager getTokenManger() {
        return new LocalTokenManager();
    }

    @ServerEndpoint(value = "/WebSocket", configurator = CallerHackConfigurator.class)
    public static class ExampleWebSocketEndPoint extends ConcreteWebSocketEndPoint {

        public ExampleWebSocketEndPoint() {
            super();
            registerClasses(ExampleApi.class);
        }
    }

    @Bean
    public ServletRegistrationBean testServlet() {
        ServletContainer container = new ServletContainer();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(
                container, "/WebSocket"){
            @Override
            public void onStartup(ServletContext servletContext) throws ServletException {
                servletContext.addListener(new ServletContextListener() {

                    @Override
                    public void contextInitialized(ServletContextEvent sce) {
                        final ServerContainer serverContainer = (ServerContainer) sce.getServletContext()
                                .getAttribute("javax.websocket.server.ServerContainer");

                        try {
                            serverContainer.addEndpoint(ExampleWebSocketEndPoint.class);
                        } catch (DeploymentException e) {
                            e.printStackTrace();
                        }
                    }

                    @Override
                    public void contextDestroyed(ServletContextEvent sce) {

                    }
                });
            }
        };
        registrationBean.setName("demo");
        registrationBean.setAsyncSupported(true);
        return registrationBean;
    }

    @Bean
    public ExampleApi getExampleApi() {
        return new ExampleImpl();
    }


    public static void main(String[] args) {
        // 设为前后端分离开发模式
//        System.setProperty("org.coodex.concrete.websocket.devMode", "true");
        SpringApplication.run(SpringBootWithoutXML.class, args);
    }

}

```

其他的都差不多了。不过在java client中，推荐使用Rx

在github上我们放了一个[archetype](https://github.com/coodex2016/concrete-archetype)，是以上过程积累下来的代码，如果需要基于concrete构建工程，可以以它为模板。后续coodex小组会持续更新模板。

