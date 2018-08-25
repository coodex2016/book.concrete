# 3.5 数据模拟

还记得[2.1](step2_1.md)中提到的`@VehicleNum`和`@INTEGER`吗，我们先看看它们能达到什么效果。

## DemoBoot

```java
    public static void main(String[] args) {
        // step 3.5
        System.setProperty("org.coodex.concrete.jaxrs.devMode", "true");
        SpringApplication.run(DemoBoot.class, args);
    }

```

http://localhost:8080/jaxrs/I/Love/Concrete/randomPlate

多刷几下看看

明明实现中return的是null，为什么会结果是随机车牌？很简单，因为加上`org.coodex.concrete.jaxrs.devMode`系统参数后，根本**不依赖实现**!!。

> #### Hint::
>
> `org.coodex.concrete.*.devMode`是服务端模拟返回结果的系统变量，目前支持的有`jaxrs`,`websocket`,`dobbo`

coodex-utilites和cocnrete提供了一些默认的[mock注解](../impl/mocker.md)，mocker体系也是一个可扩展的。我们来做一个`WeightMock`并应用到api中

因为mock是一个通用的，所以我们建一个`demo-mock`模块，目录位置在工程根目录下

增加依赖
```xml
    <dependencies>
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>coodex-utilities</artifactId>
        </dependency>
    </dependencies>
```

新建模拟注解`org.coodex.concrete.demo.WeightMock`

```java
package org.coodex.concrete.demo;

import org.coodex.pojomocker.Mock;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Mock // 声明该注解是用于mock的
public @interface WeightMock {
}
```

新建`org.coodex.concrete.demo.WeightMocker`
```java
package org.coodex.concrete.demo;

import org.coodex.pojomocker.AbstractMocker;
import org.coodex.util.Common;

public class WeightMocker extends AbstractMocker<WeightMock> {
    @Override
    public Object mock(WeightMock mockAnnotation, Class clazz) {
        if (Float.class.equals(clazz) || float.class.equals(clazz)) {
            return (float) Common.random(70.0, 140.9);
        } else if (Integer.class.equals(clazz) || int.class.equals(clazz)) {
            return Common.random(70, 140);
        } else
            return null;
    }
}
```

注册到ServiceLoader，`resources/META-INF/services/org.coodex.pojomocker.Mocker`
```
org.coodex.concrete.demo.WeightMocker
```

让api模块依赖`demo-mock`

```xml
        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>demo-mock</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
```

`Girl`类改造一下
```java
package org.coodex.concrete.demo.api.pojo;

import org.coodex.concrete.api.mockers.Name;
import org.coodex.concrete.demo.WeightMock;
import org.coodex.pojomocker.annotations.INTEGER;

public class Girl {

    private String name;
    private Integer height;
    private Integer weight;
    private Integer age;
    private Integer stars = 0;

    public Girl() {
    }

    public Girl(String name) {
        this.name = name;
    }

    @Name //随机姓名
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @INTEGER(min = 0, max = 100)
    public Integer getStars() {
        return stars;
    }

    public void setStars(Integer stars) {
        this.stars = stars;
    }

    @INTEGER(min = 150, max= 170)
    public Integer getHeight() {
        return height;
    }

    public void setHeight(Integer height) {
        this.height = height;
    }

    @WeightMock // 新建的Mock
    public Integer getWeight() {
        return weight;
    }

    public void setWeight(Integer weight) {
        this.weight = weight;
    }

    @INTEGER(min = 18, max = 36)
    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

好了，把服务跑起来。访问 http://localhost:8080/jaxrs/Girls

## mock在客户端的使用

刚才我们看到的示例是前后端之间的mock，也就是前端不依赖后端实现，那么模块间如何达到互不依赖呢？0.2.3版本开始支持Client的模拟。

我们到client-invoker模块中，做一些修改来看看效果。

我们之前的client-invoker都是依赖服务端的，现在把服务端停掉，增加客户端mock的配置

以GirlsServiceExample为例，这个实现中，我们使用的是demo模块，我们让客户端调用demo模块时使用模拟数据的方式，resources下建一个`moduleMock.properties`
```properties
client.demo=true
```
因为设调用demo模块为mock以后，不再进行到demo模块的服务端访问，所以，控制台不会再有输出，为了看到演示效果，我们把GirlServiceExample改造一下

```java
package org.coodex.concrete.demo.invoker;

import com.alibaba.fastjson.JSON;
import org.coodex.concrete.Client;
import org.coodex.concrete.demo.api.GirlService;
import org.coodex.concrete.demo.api.pojo.Girl;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class GirlServiceExample {
    private final static Logger log = LoggerFactory.getLogger(GirlServiceExample.class);

    private static <T> T trace(T object){
        log.debug(JSON.toJSONString(object));
        return object;
    }

    public static void main(String [] args){
        GirlService girlService = Client.getInstance(GirlService.class, "demo");

        String [] girls = new String[]{"Lele", "Feifei", "Qingqing", "Yanyan", "Davidoff"};

        for(String name: girls){
            girlService.saveGirl(new Girl(name));
        }
        Girl davidoff = trace(girlService.get("Davidoff"));
        davidoff.setAge(41);
        girlService.updateGirlInfo("Davidoff", davidoff);
        trace(girlService.get());
        girlService.deleteByName("Davidoff");
        trace(girlService.get());
        Girl girl = trace(girlService.get("Lele"));
        girl.setStars(100);
        girlService.updateGirlInfo("Lele", girl);
        trace(girlService.getStars("Lele"));
    }
}
```

run一下看看。

> #### Hint::
>
> 因为mock配置只是跟开发有关，所以不要把moduleMock.properties提交到代码库中

## 意义何在？

数据模拟在协作开发时还是蛮有价值的，把mock注解加到服务接口和pojo上，模块间不同的开发者不再依赖别人的实现，前后端分离开发时，也不需要前端等后端。

举个例子，我们把团队分为三队人马，一开始大家一起协定api，然后api组狂写mock，前端做交互，后端做实现，为了支持三伙人并行开工，单建一个前后端分离的发布模块。
因为concrete自带了各种默认的mock，所以这么开工互不影响，api组提交一次完善了mock的api（最好在加上自动部署），前端看到的数据就会(看起来)更真实一些。


