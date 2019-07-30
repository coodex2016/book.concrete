# concrete使用

本章主要由浅入深的介绍concrete使用。concrete的推荐实践需要开发者掌握一定的maven知识和S.O.L.I.D.原则的理解。

虽然concrete并不依赖spring，但是我们依然推荐使用spring，spring-boot/spring-data实在是效率利器。因此，我们的示例中会使用到他们。

concrete的物料清单(concrete-bom)中，已经把spring-framework/ spring-boot/ spring-data引入进来了，方便大家使用。

在开始使用之前，我们先建一个maven工程，并定立一下规约：

- 演示模块的groupId为`org.coodex.concrete.demo`
- 演示包名为`org.coodex.concrete.demo.**`
  - 例外：如使用到`coodex-mock`的`Assignation`功能时，包`mock.assign`遵循`concrete-mock`规范
- 用于定义concrete service的包根据作用不同，可以定义为`org.coodex.concrete.demo.**.api`或`org.coodex.concrete.demo.**.rpc`
- 实现统一放到`org.coodex.concrete.demo.**.impl`中，并遵循`javax.inject`规范
- 模块根据用途划分，最起码分为`**-api`、`**-impl`、`**-boot`三个模块，原因：
  - 各司其职，在模块级别达到到单一职责原则，也有利于团队协作分工
  - `**-api`负责定义`service`，除发布服务以外，还可以供调用者使用，而调用者并不依赖实现
  - `**-impl`中对定义的服务进行实现，只需要考虑如何达到服务能力即可，不需要关注如何发布服务
  - `**-boot`，使用[spring-boot](https://spring.io/projects/spring-boot)发布服务，负责最后的组装

好的，我们开始。

## 建立演示工程，将concrete物料清单放入到依赖管理中

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.coodex.concrete.demo</groupId>
    <artifactId>demo-pom</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>demo-api</module>
    </modules>

    <properties>
        <concrete.version>0.4.0-SNAPSHOT</concrete.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <!-- 使用java8的-parameters编译参数，可以保留参数名 -->
                    <compilerArgument>-parameters</compilerArgument>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

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

</project>
```

`concrete-bom`中定义了`concrete**`、`coodex**`、`spring**`等常用的库和框架，减少构建者管理项目依赖的复杂度

[next, 定义一个concrete服务](01.defineService.md)