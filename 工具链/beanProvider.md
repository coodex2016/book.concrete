# BeanProvider

考虑到[Spring](http://spring.io)的强大，concrete实现了基于Spring的BeanProvider。


```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-core-spring</artifactId>
        </dependency>
```


```xml
    <bean class="org.coodex.concrete.spring.SpringBeanProvider"></bean>
```

当同一个服务模块有多个实现时，可以通过实现ConflictSolution解决冲突。默认提供了按照名称的解决方案。

-----------------------------
TODO: 需要考虑基于Guice的BeanProvider
