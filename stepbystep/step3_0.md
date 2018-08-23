# 3.0 服务命名及参数组合

> #### Info::
>
> 仅限jaxrs发布服务有效

还记得示例中丑陋的`OrgCoodexConcreteDemoApiDemoService`吗？这一小节我们尝试通过一些定义来让concrete提供的服务更贴近Restful规范。

## 服务命名

设置[`@MicroService`](../definition/MicroService.md)注解的value即可，在本例中，我们该几个地方

```java
@Description(
        name = "demo模块",
        description = "本模块主要通过一些简单的示例，向大家介绍[concrete](https://concrete.coodex.org)的功能。blablabla..."
)
// step 3.0: 定义服务模块名
@MicroService("I/love/concrete")
public interface DemoService extends ConcreteService {

    @Description(
            name = "求两数之和",
            description = "不多说，都会"
    )
    // step 3.0: 定义接口名
    @MicroService("andTeachMe/How/Much/is/{x1}/plus/{x2}/plz")
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

运行DemoBoot，然后在浏览器里访问 http://localhost:8080/jaxrs/I/Love/Concrete/andTeachMe/How/Much/is/1/plus/2/plz

好像更麻烦了呃，再来解析一下
- http://localhost:8080/jaxrs 之前一样
- I/Love/Concrete <-- DemoService的注解内容，注意一下，concrete默认规则是把每一节都转成大驼峰
- andTeachMe/How/Much/is/1/plus/2/plz <-- add接口上的注解内容，其中\{x1\}\{x2\}用来指定PathParameter

好了，不开玩笑，我们再来编写一个更贴近Restful风格的服务接口

在`org.coodex.concrete.demo.api`下

```java
package org.coodex.concrete.demo.api;

import org.coodex.concrete.api.ConcreteService;
import org.coodex.concrete.api.MicroService;
import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.util.Parameter;

import java.util.List;

@MicroService("Girls")
public interface GirlService extends ConcreteService {

    List<Girl> get();

    Girl get(@Parameter("name") String name);

    @MicroService
    void saveGirl(@Parameter("girl") Girl girl);

    @MicroService
    void deleteByName(@Parameter("name") String name);

    @MicroService("{name}")
    void updateGirlInfo(@Parameter("name") String name,
                        @Parameter("girl") Girl girl);

    @MicroService("{name}/stars")
    Integer getStars(@Parameter("name") String name);
}
```

`org.coodex.concrete.demo.api.pojo.Girl`
```java
package org.coodex.concrete.demo.api.pojo;

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

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getStars() {
        return stars;
    }

    public void setStars(Integer stars) {
        this.stars = stars;
    }

    public Integer getHeight() {
        return height;
    }

    public void setHeight(Integer height) {
        this.height = height;
    }

    public Integer getWeight() {
        return weight;
    }

    public void setWeight(Integer weight) {
        this.weight = weight;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

用之前的`XGen`生成文档看一下，我们多了一个/Girls服务模块
- /Girls 有GET和POST方法
- /Girls/{name} 有GET/PUT/DELETE方法
- /Girls/{name}/stars 一个GET方法

针对上例，我们按照Restful风格逐一说明一下
- /Girls GET，即获得全部资源
- /Girls POST，新建一个资源
- /Girls/{name} GET，获取一个指定的资源
- /Girls/{name} PUT，修改一个指定的资源
- /Girls/{name} DELETE，删除一个资源
- /Girls/{name}/stars GET，获取一个资源的一项属性

很贴近Restful风格了，对吧，23333

怎么做到的？我们一个个看
- /Girls GET，即获得全部资源
    对应`List<Girl> get();`，本来有个get接口名，在没有使用`@MicroService`注解的接口上，jaxrs模块会自动抹去谓词
    
- /Girls POST，新建一个资源
    对应`@MicroService void saveGirl(@Parameter("girl") Girl girl);`，一样的，使用`@MicroService()`声明抹去接口名，
    根据[谓词规则](../impl/jsr311.md#谓词定义)使用POST方法
    
- /Girls/{name} GET，获取一个指定的资源
    对应`Girl get(@Parameter("name") String name);`，根据[谓词规则](../impl/jsr311.md#谓词定义)使用GET方法，
    在没有使用`@MicroService`注解的接口上，jaxrs模块会自动抹去谓词
    
- /Girls/{name} PUT，修改一个指定的资源
    对应`@MicroService("{name}") void updateGirlInfo(@Parameter("name") String name, @Parameter("girl") Girl girl);`，
    根据[谓词规则](../impl/jsr311.md#谓词定义)使用PUT方法
    
- /Girls/{name} DELETE，删除一个资源
    对应`@MicroService void deleteByName(@Parameter("name") String name);`，
    根据[谓词规则](../impl/jsr311.md#谓词定义)使用DELETE方法，
    使用`@MicroService`声明后，把接口名抹去了
    
- /Girls/{name}/stars GET，获取一个资源的一项属性
    对应`@MicroService("{name}/stars") Integer getStars(@Parameter("name") String name);`，使用`@MicroService`重载了接口名

先做一个实现
```java
package org.coodex.concrete.demo.impl;

import org.coodex.concrete.common.IF;
import org.coodex.concrete.demo.api.GirlService;
import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.concrete.demo.impl.copier.GirlCopier;

import javax.inject.Inject;
import javax.inject.Named;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Named
public class GirlServiceImpl implements GirlService {

    @Inject
    private GirlCopier girlCopier;

    private Map<String, Girl> girlsMap = new HashMap<String, Girl>();

    @Override
    public List<Girl> get() {
        return new ArrayList<Girl>(girlsMap.values());
    }

    @Override
    public Girl get(String name) {
        return IF.isNull(girlsMap.get(name), "Girl not found");
    }

    @Override
    public void saveGirl(Girl girl) {
        IF.is(girlsMap.containsKey(girl.getName()), "Girl exists.");
        girlsMap.put(girl.getName(), girl);
    }

    @Override
    public void deleteByName(String name) {
        girlsMap.remove(name);
    }

    @Override
    public void updateGirlInfo(String name, Girl girl) {
        Girl g = get(name);
        girlCopier.copy(girl, g);
    }

    @Override
    public Integer getStars(String name) {
        return get(name).getStars();
    }
}
```

里面用到了一个copier，后续在详细介绍，代码列上
```java
package org.coodex.concrete.demo.impl.copier;

import org.coodex.concrete.common.AbstractCopier;
import org.coodex.concrete.demo.api.pojo.Girl;

import javax.inject.Named;

@Named
public class GirlCopier extends AbstractCopier<Girl,Girl> {
    @Override
    public Girl copy(Girl src, Girl target) {
        target.setName(src.getName());
        if(src.getHeight() != null)
            target.setHeight(src.getHeight());
        if(src.getAge() != null)
            target.setAge(src.getAge());
        if(src.getWeight() != null)
            target.setWeight(src.getWeight());
        if(src.getStars() != null)
            target.setStars(src.getStars());
        return target;
    }
}
```

DemoBoot里，注册上这个服务接口。

> #### Hint::
>
> 因为以后我们会有更多的服务需要被注册进来，所以，选择按package注册的方法，以后再增加服务时DemoBoot就不需要动了
```java
public static class JaxRSApplication extends ConcreteJSR339Application {
        public JaxRSApplication() {
            register(
                    // 使用 jackson 作为 jaxrs的序列化和反序列化实现
                    JacksonFeature.class);

            // 按照包名约定注册服务
            registerPackage(DemoService.class.getPackage().getName());
        }
    }
```

然后在客户端调用模块写一些调用示例

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
        </dependency>
```
concrete默认选择fastjson作为pojo的序列化工具


```java
package org.coodex.concrete.demo.invoker;

import org.coodex.concrete.Client;
import org.coodex.concrete.demo.api.GirlService;
import org.coodex.concrete.demo.api.pojo.Girl;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class GirlServiceExample {
    private final static Logger log = LoggerFactory.getLogger(GirlServiceExample.class);

    public static void main(String [] args){
        GirlService girlService = Client.getInstance(GirlService.class, "demo");

        String [] girls = new String[]{"Lele", "Feifei", "Qingqing", "Yanyan", "Davidoff"};

        for(String name: girls){
            girlService.saveGirl(new Girl(name));
        }
        Girl davidoff = girlService.get("Davidoff");
        davidoff.setAge(41);
        girlService.updateGirlInfo("Davidoff", davidoff);
        girlService.get();
        girlService.deleteByName("Davidoff");
        girlService.get();
        Girl girl = girlService.get("Lele");
        girl.setStars(100);
        girlService.updateGirlInfo("Lele", girl);
        girlService.getStars("Lele");
    }
}
```

如果你的log4j.properties还没改的话，那么就可以在控制台看到请求和响应的信息了

再跑一下看看，咦，出错了呢。我们看一下控制台输出信息：

```
Exception in thread "main" org.coodex.concrete.client.jaxrs.JaxRSClientException: |UNKNOWN ERROR|: Girl exists..
	at org.coodex.concrete.client.jaxrs.JaxRSInvoker.throwException(JaxRSInvoker.java:98)
	at org.coodex.concrete.client.jaxrs.JaxRSInvoker.processResult(JaxRSInvoker.java:114)
	at org.coodex.concrete.client.jaxrs.JaxRSInvoker.execute(JaxRSInvoker.java:228)
	at org.coodex.concrete.client.impl.AbstractSyncInvoker$1$1.proceed(AbstractSyncInvoker.java:59)
	at org.coodex.concrete.core.intercept.SyncInterceptorChain$MethodInvocationChain.proceed(SyncInterceptorChain.java:179)
	at org.coodex.concrete.core.intercept.AbstractSyncInterceptor.around(AbstractSyncInterceptor.java:69)
	at org.coodex.concrete.core.intercept.AbstractSyncInterceptor.invoke(AbstractSyncInterceptor.java:49)
	at org.coodex.concrete.core.intercept.SyncInterceptorChain.invoke(SyncInterceptorChain.java:69)
	at org.coodex.concrete.client.impl.AbstractSyncInvoker$1.call(AbstractSyncInvoker.java:53)
	at org.coodex.closure.AbstractClosureContext.closureRun(AbstractClosureContext.java:47)
	at org.coodex.closure.StackClosureContext.call(StackClosureContext.java:70)
	at org.coodex.concrete.common.ConcreteContext.runWithContext(ConcreteContext.java:154)
	at org.coodex.concrete.client.impl.AbstractSyncInvoker.invoke(AbstractSyncInvoker.java:50)
	at org.coodex.concrete.client.impl.JavaProxyInstanceBuilder$1.invoke(JavaProxyInstanceBuilder.java:47)
	at com.sun.proxy.$Proxy3.saveGirl(Unknown Source)
	at org.coodex.concrete.demo.invoker.GirlServiceExample.main(GirlServiceExample.java:18)
```

`Girl exists..`，concrete的IF断言提示的异常信息，这种用法并不规范，到[3.4 异常信息](step3_4.md)我们再详细说明。

截止到当前的代码： https://github.com/coodex2016/concrete-demo/tree/step3_0

## 参数组合

在上例中，我们看到了POST pojo，那么原始类型及String如何post呢？

我们回到api模块来做一个示例，先加上concrete-jaxrs的依赖
```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs</artifactId>
            <scope>provided</scope>
        </dependency>
```
DemoService，修改`add`和`sayHello`
```java
    // step 3.0: 定义接口名
    // step 3.0.1: 演示参数组合
    // @MicroService("andTeachMe/How/Much/is/{x1}/plus/{x2}/plz")
    int add(
            //step 3.0.1:参数组合
            @Body
            @Description(name = "被加数")
            @Parameter("x1") int x1,
            //step 3.0.1:参数组合
            @Body
            @Description(name = "加数")
            @Parameter("x2") int x2);

    @Description(
            name = "就是个示意，编不出来了 :("
    )
    String sayHello(
            // step 3.0.1: post数据
            @Body
            @Description(name = "要say hello的名字")
            @Parameter("name") String name);
```

用XGen生成jquery脚本，拷贝到`resources/static/jquery`下，运行DemoBoot

http://localhost:8080/jquery/index.html

打开浏览器的调试工具，切换到网络栏。点这两个按钮，看看跟之前请求有什么变化？

截止到当前的代码： https://github.com/coodex2016/concrete-demo/tree/step3_0_1

> #### Hint::
>
> 示例中，我们使用的是基础类型，如果参数可能为空，那么我们要注意两点
> - 避免把参数放到path中
> - 使用包装类型
