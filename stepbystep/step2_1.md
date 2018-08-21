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

VehiclePlate.java
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

