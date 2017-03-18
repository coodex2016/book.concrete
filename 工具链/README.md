# Concrete工具链

Concrete工具链提供了一套Concrete规范的参考实现。

## 使用

建立maven工程，导入concrete的pom
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-bom</artifactId>
            <version>0.2.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

编译插件推荐如下设置
```xml
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
```