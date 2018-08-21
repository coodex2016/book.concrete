# 单元调试支持

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-test</artifactId>
            <scope>test</scope>
        </dependency>
```

## 使用

单元调试时，所有的TestCase应继承自ConcreteTestCase

```java
public class SomeTestCase extends ConcreteTestCase{
    
}
```

## @TokenID

用以模拟服务原子运行时所需的令牌信息。

value: 默认随机，可以指定id

例如

```java
public class A extends ConcreteTestCase {


    private Token token = TokenWrapper.getInstance();

    @Test
    @TokenID("1")
    public void test() {
        token.setAttribute("test", "TOKEN");
    }

    @Test
    @TokenID("1")
    public void test2() {
        System.out.println(token.getAttribute("test"));// TOKEN
    }

    @Test
    public void test3() {
        System.out.println(token.getAttribute("test"));// null
    }
}
```
