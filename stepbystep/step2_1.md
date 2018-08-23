# 2.1 生成文档

[文档化](../definition/Description.md)是concrete的一个重要规范。

我们回到step1的代码，api部分，开始增加文档化相关的内容。

```java
package org.coodex.concrete.demo.api;

import org.coodex.concrete.api.ConcreteService;
import org.coodex.concrete.api.Description;
import org.coodex.concrete.api.MicroService;
import org.coodex.concrete.demo.api.pojo.VehiclePlate;
import org.coodex.util.Parameter;

@Description(
        name = "demo模块",
        description = "本模块主要通过一些简单的示例，向大家介绍[concrete](https://concrete.coodex.org)的功能。blablabla..."
)
@MicroService()
public interface DemoService extends ConcreteService {

    @Description(
            name = "求两数之和",
            description = "不多说，都会"
    )
    int add(
            @Description(name = "被加数")
            @Parameter("x1") int x1,
            @Description(name = "加数")
            @Parameter("x2") int x2);

    @Description(
            name = "就是个示意，编不出来了 :("
    )
    String sayHello(
            @Description(name = "要say hello的名字")
            @Parameter("name") String name);

    @Description(
            name = "获取一个车牌",
            description = "一会演示mock用"
    )
    VehiclePlate randomPlate();

}
```

我们增加了一个接口，用来演示pojo的文档化，pojo的class是`org.coodex.concrete.demo.api.pojo.VehiclePlate`

```java
package org.coodex.concrete.demo.api.pojo;

import org.coodex.concrete.api.Description;
import org.coodex.concrete.api.mockers.VehicleNum;
import org.coodex.pojomocker.annotations.INTEGER;

public class VehiclePlate {

    @VehicleNum
    private String code;

    @INTEGER(range = {0, 1, 2, 3, 4, 9})
    private Integer color;

    @Description(name = "车牌号码")
    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    @Description(name = "车牌颜色",
    description = "代码参照国家标准, 0: 白; 1: 黄; 2: 蓝; 3: 黑; 4: 绿; 9: 其他")
    public Integer getColor() {
        return color;
    }

    public void setColor(Integer color) {
        this.color = color;
    }
}
```
> #### Hint::
>
> `@VehicleNum` 和 `@INTEGER` 是mock的注解，先放着，一会用  
> `@Description` 用来修饰bean property,推荐在get和is方法上进行修饰，继承类可以重载

把实现补上，return 个 null 先

```java
    @Override
    public VehiclePlate randomPlate() {
        return null;
    }
```

ok, 现在工程build正常，开始生成文档

> #### Hint::
>
> 注意，不是必须现有实现才能生成文档，我们的理念是，前后端、系统间、模块间只依赖api，因此只要有api即可生成文档。  

考虑到模块间可能组合提供服务，我们单独建一个maven子模块`api-helper`来负责产生这些文档和脚本,`api-helper`放在工程根目录下，以示通用

在pom里增加如下依赖
```xml
    <dependencies>

        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-api-tools</artifactId>
        </dependency>

        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>demo-api</artifactId>
            <version>${project.parent.version}</version>
        </dependency>


    </dependencies>
```


> #### Hint::
>
> [concrete-api-tools](../impl/API.md)是concrete生成文档、脚本、代码的工具包  
> 其他api模块可以后续逐一加入

新建class `org.coodex.concrete.generators.XGen`

```java
package org.coodex.concrete.generators;

import org.coodex.concrete.apitools.API;
import org.coodex.concrete.demo.api.DemoService;

import java.io.IOException;

public class XGen {

    public static void main(String [] args) throws IOException {

        API.generateFor("demo.doc",
                DemoService.class.getPackage().getName());
    }
}
```

> #### Hint::
>
> 说明一下，`org.coodex.concrete.apitools.API` 提供了两个静态接口，`generate`和`generateFor`，差别是：
> - generate: 硬编码指定渲染描述符和生成的路径
> - generateFor: 通过concrete的[配置规范](../impl/config.md#concrete-api-tools)配置渲染描述符和生成的路径
>
> 本例中，工程的`.gitignore`忽略了 `**/resources/api_gen.properties` 和 `**/resources/api_gen.*.properties`，所以，请各位自行在resources目录下新建配置文件。忽略这些文件的原因是，我们在协作过程中，个人的工作环境可能不一致，所以，大家按照自己的环境配置，互相之间不会影响

api_gen.properties
```properties
demo.doc.desc=JaxRS.doc.backend.gitbook.v1
demo.doc.path=$$$YOUR PATH, Change it$$$
```

or api_gen.demo.doc.properties
```properties
desc=JaxRS.doc.backend.gitbook.v1
path=$$$YOUR PATH, Change it$$$
```

run一下，出错估计就是路径或者目录权限的问题，自行解决

进入你指定的路径

```
gitbook install
gitbook serve
```

然后打开 http://localhost:4000 

> #### Hint::
>
> 对了，记得先装好 `gitbook-cli`， `npm install gitbook-cli -g`
