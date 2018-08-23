# 3.1 RBAC

RBAC，role based access control，基于角色的访问控制。

既然是基于角色的，就需要有角色的载体，在concrete里，账户就是角色的载体，一个账户可以有多个角色。

## 账户体系

concrete的账户体系是一个可扩展的体系，大致结构如下：

- 聚合账户工厂
    - 根据账户ID从多个已注册的账户工厂中选择适用，如果是单一账户体系的话，从这开始
        - 创建账户
            - 账户 1 --> * 角色

> #### Hint:: 关于聚合账户工厂
>
> 同一个系统有多套账户体系的情况也还是蛮多的，使用concrete时，我们可以把账户体系分类通过账户的id类型预先区分出来，然后由各自的账户工厂管理账户。
> 我们只需要把这些工厂都注册到依赖注入环境中，使用聚合账户工厂来适配即可。
> 不仅如此，这样做还有利于不同账户体系的复用。
>
> 假设我们有三种用户(账户)体系，互联网的，组织结构的，运维管理的，那么，我们把这三种分别独立实现，系统需要哪几个，依赖进来就行了。
> 推荐使用[NamedAccount](https://github.com/coodex2016/concrete.coodex.org/blob/0.2.x/01.spec/concrete-api/src/main/java/org/coodex/concrete/common/NamedAccount.java)
>
> concrete的账户体系规范只是一套接口，我们还可以通过Bridge或者Adaptor模式是复用已有的用户帐号体系实现。
>
> 为了支持快速开发，concrete提供了一套[简单账号](../accounts/README.md)。我们依此进行演示。

## 定义RBAC

回到RBAC上，对于一套系统而言，在需求不变的情况下，每个业务的开展所需要的角色其实是确定的，所以，我们在分析阶段就可以把它们分析出来，
在设计阶段，直接编写到设计定义中。

我们先在DemoService中简单体验一下

```java
package org.coodex.concrete.demo.api;

import org.coodex.concrete.api.AccessAllow;
import org.coodex.concrete.api.ConcreteService;
import org.coodex.concrete.api.Description;
import org.coodex.concrete.api.MicroService;
import org.coodex.concrete.demo.api.pojo.VehiclePlate;
import org.coodex.util.Parameter;

@Description(
        name = "demo模块",
        description = "本模块主要通过一些简单的示例，向大家介绍[concrete](https://concrete.coodex.org)的功能。blablabla..."
)
@MicroService("I/love/concrete")
public interface DemoService extends ConcreteService {

    @Description(
            name = "求两数之和",
            description = "不多说，都会"
    )
    // step 3.1
    @AccessAllow
    int add(
            @Description(name = "被加数")
            @Parameter("x1") int x1,
            @Description(name = "加数")
            @Parameter("x2") int x2);

    @Description(
            name = "就是个示意，编不出来了 :("
    )
    // step 3.1
    @AccessAllow(roles = "A")
    @Safely
    String sayHello(
            @Description(name = "要say hello的名字")
            @Parameter("name") String name);

    @Description(
            name = "获取一个车牌",
            description = "一会演示mock用"
    )
    // step 3.1
    @AccessAllow(roles = {"A", "B"})
    VehiclePlate randomPlate();

}
```

上例中，我们针对三个接口都指定了AccessAllow，但是roles不同，现在逐一说明一下：
- `@AccessAllow`，表明只要是登录的帐号，并且是有效的就行
- `@AccessAllow(roles = "A") @Safely`，表明需要当前帐号必须是可信的，还得有A的角色
- `@AccessAllow(roles = {"A", "B"})`，表明需要当前帐号得有A或者B的角色

> #### Hint::有效、可信
>
> - 有效，指账户可用，参见 https://github.com/coodex2016/concrete.coodex.org/blob/0.2.x/01.spec/concrete-api/src/main/java/org/coodex/concrete/common/Account.java#L40-L45
> - 可信，指当前会话是否可信，参见https://github.com/coodex2016/concrete.coodex.org/blob/0.2.x/01.spec/concrete-api/src/main/java/org/coodex/concrete/common/Token.java#L68-L73
>
> 有效应该好理解，是账户的属性，比如说，某人离职了，那么他的帐号在移除之前，就不是有效的
>
> 可信的，是指当前会话的用户是否可信，举个例子，大家使用淘宝的时候，登录过以后，一段时间再打开，在界面上还是显示你的头像和名字，也就是说，
> 当前会话的用户还是你，但是，当你需要查看自己的订单时，系统会要求你登录，因为从cookie也好，localStorage也好，
> 获取到的信息在服务端并没有可信任的登录信息，所以需要登录来让本次会话可信，通过这种机制来提高数据的安全性。

好的，生成文档看看，角色acl已经写到文档里了。

我们给DemoBoot增加上一个RBAC的[拦截器](../impl/interceptor.md)，让RBAC功能启用
```java
    /**
     * 注册RBAC的拦截器
     * @return
     */
    @Bean
    public RBACInterceptor rbacInterceptor(){
        return new RBACInterceptor();
    }
```

> #### Hint::
>
> 在xml注册也是一样的


DemoBoot运行起来，访问以下三个链接
- http://localhost:8080/jaxrs/I/Love/Concrete/add/1/2
- http://localhost:8080/jaxrs/I/Love/Concrete/sayHello/davidoff
- http://localhost:8080/jaxrs/I/Love/Concrete/randomPlate

我们看到的结果都是code为1005的一个json，返回的msg不友好，再忍忍，到[3.4 异常信息](step3_4.md)细说。

开调试工具，我们可以看到，http响应码是400。

> #### Comment::
>
> 很多人喜欢用正常的响应码（2xx）通过errorCode（0-成功，其他失败）来传递数据，这个做法是不规范的，
> http协议的响应码已经很规范了，我们还是应该遵循已有的规范。

## 使用简单账户体系来演示

我们先规划一下，我们需要用到以下几个账户来演示

| 账户 | 角色 | add | sayHello | randomPlate |
| ---- | ---- | --- | --- | --- |
| Lele | A, B | true | true | true |
| Feifei | B, C | true | false | true |
| Nana | X | true | false | false |

在发布模块中，增加简单账户体系的依赖

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-accounts-simple-impl</artifactId>
        </dependency>
```

`resouces/accounts`下建三个文件

Lele.properties
```properties
name=Lele
roles=A, B
password=123456
# base32
authKey=1234567890
```

Feifei.properties
```properties
name=Feifei
roles=B, C
password=123456
# base32
authKey=1234567890
```

Nana.properties
```properties
name=Nana
roles=X
password=123456
# base32
authKey=1234567890
```

修改DemoBoot：注册简单账户的登录服务实现；注册简单账户工厂；发布时注册简单账户登录服务
```java
    /**
     * step 3.1 注册简单账户工厂
     * @return
     */
    @Bean
    public AccountFactory accountFactory(){
        return new SimpleAccountFactory();
    }
    
    /**
     * step 3.1 注册简单账户的登录服务实现
     * @return
     */
    @Bean
    public Login login(){
        return new SimpleAccountLoginImpl();
    }

    /**
     * jsr339(jaxrs 2.0)规范的Application
     */
    public static class JaxRSApplication extends ConcreteJSR339Application {
        public JaxRSApplication() {
            register(
                    // 使用 jackson 作为 jaxrs的序列化和反序列化实现
                    JacksonFeature.class,
                    // step 3.1 简单账户登录服务
                    Login.class);

            // 按照包名约定注册服务
            registerPackage(DemoService.class.getPackage().getName());
        }
    }
```

`SimpleAccountLoginImpl`默认开启了TOTP双因子认证，为了演示方便，把它关掉

resources下新建一个simpleAccounts.properties
```properties
authCode=false
```

好了，我们开始准备调试案例，直接使用junit做测试用例，使用调用服务的方式调试。下一步详解如何进行服务端[单元调试](step3_2.md)。

client-invoker模块中，增加junit依赖
```xml
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-accounts-simple-api</artifactId>
            <scope>test</scope>
        </dependency>
```

test作用域中编写用例 `test.org.coodex.concrete.demo.test.RBACTest`
```java
package test.org.coodex.concrete.demo.test;

import org.coodex.concrete.Client;
import org.coodex.concrete.accounts.simple.api.Login;
import org.coodex.concrete.common.ConcreteException;
import org.coodex.concrete.demo.api.DemoService;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

public class RBACTest {

    private Login login;
    private DemoService demoService;

    @Before
    public synchronized void init(){
        if(login == null){
            login = Client.getInstance(Login.class, "demo");
            demoService = Client.getInstance(DemoService.class, "demo");
        }
    }

    /**
     * @param user
     * @param add add方法是否可用
     * @param sayHello sayHello方法是否可用
     * @param randomPlate randomPlate方法是否可用
     */
    private synchronized void check(String user, boolean add, boolean sayHello, boolean randomPlate){
        login.login(user,"123456",null);
        try {
            try {
                demoService.add(1, 2);
                Assert.assertTrue(add);
            } catch (ConcreteException e) {
                Assert.assertTrue(!add);
            }

            try {
                demoService.sayHello(user);
                Assert.assertTrue(sayHello);
            } catch (ConcreteException e) {
                Assert.assertTrue(!sayHello);
            }

            try {
                demoService.randomPlate();
                Assert.assertTrue(randomPlate);
            } catch (ConcreteException e) {
                Assert.assertTrue(!randomPlate);
            }
        }finally {
            login.logout();
        }
    }

    @Test
    public void testLele(){
        check("Lele", true, true, true );
    }

    @Test
    public void testFeifei(){
        check("Feifei", true, false, true );
    }

    @Test
    public void testNama(){
        check("Nana", true, false, false );
    }
}
```

 junit也能看到后台的输出，大家可以自行修改修改进行实验。
 
 截止到此阶段的代码： https://github.com/coodex2016/concrete-demo/tree/step3_1