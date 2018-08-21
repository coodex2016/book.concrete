# 实现api

建一个 `demo-api` 子模块，模块目录为`demo/api`

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



