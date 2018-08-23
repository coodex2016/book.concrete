# 3.2 单元调试

`concrete-test` 基于junit提供了一套针对ConcreteService实现的单元调试方案，本小节我们学习它的使用。

> #### Hint::
>
> 先说一下，单元调试，属于实现部分的配套内容，是开发者的任务，不是测试人员的任务。

我们还是按照[3.1](step3_1.md)的定义来进行DemoService的单元调试，只不过，这次不发布服务。并且，这次我们体验一下`@Safely`

在demo-impl模块中，增加以下依赖，注意一下，作用域是test，按照我们之前的约定，应该放在依赖的最上面
```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core-spring</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-accounts-simple-impl</artifactId>
            <scope>test</scope>
        </dependency>
```

- 依赖concrete-accounts-simple-impl是为了提供账户系统

我们预期的结果是

| 账户 | 角色 | add | sayHello | randomPlate |
| ---- | ---- | --- | --- | --- |
| Lele | A, B | true | 信任: true; 不信任: false | true |
| Feifei | B, C | true | false | true |
| Nana | X | true | false | false |

新建调试单元`test.org.coodex.concrete.demo.test.DemoServiceTest`
```java
package test.org.coodex.concrete.demo.test;

import org.coodex.concrete.accounts.AccountIDImpl;
import org.coodex.concrete.common.*;
import org.coodex.concrete.demo.api.DemoService;
import org.coodex.concrete.test.ConcreteTestCase;
import org.coodex.concrete.test.TestSubjoin;
import org.coodex.concrete.test.TestSubjoinItem;
import org.coodex.concrete.test.TokenID;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.inject.Inject;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:demoService.xml")
public class DemoServiceTest extends ConcreteTestCase {
    @Inject
    private AccountFactory accountFactory;

    @Inject
    private DemoService demoService;

    @Inject
    private Token token;

    private void setUser(String userName, boolean credible) {
        token.setAccount(accountFactory.getAccountByID(new AccountIDImpl(99, userName)));
        token.setAccountCredible(credible);
    }

    private void check(String userName, boolean add, boolean sayHello, boolean randomPlate) {
        try {
            Assert.assertEquals(3, demoService.add(1, 2));
            Assert.assertTrue(add);
        } catch (ConcreteException t) {
            Assert.assertFalse(add);
        }

        try {
            Assert.assertEquals(
                    String.format("Hello %s!", userName),
                    demoService.sayHello(userName));
            Assert.assertTrue(sayHello);
        } catch (ConcreteException t) {
            Assert.assertFalse(sayHello);
        }

        try {
            Assert.assertNull(demoService.randomPlate());
            Assert.assertTrue(randomPlate);
        } catch (ConcreteException t) {
            Assert.assertFalse(randomPlate);
        }
    }

    private void testUser(String userName, boolean credible,
                          boolean add, boolean sayHello, boolean randomPlate) {
        setUser(userName, credible);
        check(userName, add, sayHello, randomPlate);
    }

    @Test
    @TokenID("1")
    @TestSubjoin(@TestSubjoinItem(key = "key", value = "ok"))
    public void testLele() {
        testUser("Lele",
                true, true, true, true);
        // 注意上下两段的第三个boolean的差别
        testUser("Lele",
                false, true, false, true);
    }

    @Test
    @TokenID("2")
    public void testFeifei() {
        testUser("Feifei", true, true, false, true);
        testUser("Feifei", false, true, false, true);
    }

    @Test
    @TokenID("3")
    public void testNana() {
        testUser("Nana", true, true, false, false);
        testUser("Nana", false, true, false, false);
    }

    @Test
    @TokenID("1")
    public void testUserLele() {
        Account account = token.currentAccount();
        if (account instanceof NamedAccount) {
            Assert.assertEquals(((NamedAccount) account).getName(), "Lele");
        } else {
            Assert.assertFalse(true);
        }
    }
}
```

我们这次使用xml来定义bean，在test作用域的resources下，建一个demoService.xml，与调试单元中的`@ContextConfiguration("classpath:demoService.xml")`对应起来
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    <bean class="org.coodex.concrete.spring.ConcreteSpringConfiguration"/>
    <bean class="org.coodex.concrete.core.intercept.RBACInterceptor"/>
    <bean class="org.coodex.concrete.accounts.simple.impl.SimpleAccountFactory"/>
    <bean class="org.coodex.concrete.demo.impl.DemoServiceImpl"/>
</beans>
```
里面只用定义了我们要用到的东西

再把3.1的accounts复制到test作用域的resources下

ok，全部通过。大家可以试着改改看看结果。

上例中，我们看到几个注解，下面逐一解说一下

- @TokenID

用来指定该调试方法使用那个Token，value相同的就是一个Token，在本例中，
testLele和testUserLele就是同一个token，所以当前用户是一个

- @TestSubjoin

用来传递Subjoin，我们在testLele方法中设置了一个@TestSubjoinItem，键为key,值为ok。
我们到在DemoServiceImpl中看看
```java
    // step 3.2
    @Inject
    private Token token;
    @Inject
    private Subjoin subjoin;

    @Override
    public int add(int x1, int x2) {
        System.out.println(String.format("%s, %s",
                token.currentAccount().getId().serialize(),
                subjoin.get("key")));
        return x1 + x2;
    }
```

再跑一下看看。

截止到此阶段的代码： https://github.com/coodex2016/concrete-demo/tree/step3_2