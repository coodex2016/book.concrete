# 3.8 APM，应用性能管理


APM, Application Performance Management, 应用性能管理。

现代APM体系，基本都是参考Google的Dapper（大规模分布式系统的跟踪系统）的体系来做的。
通过跟踪请求的处理过程，来对应用系统在前后端处理、服务端调用的性能消耗进行跟踪。

业内也有不少的开源项目，concrete对APM体系进行了抽象，提炼了通用的接口和机制，
并基于zipkin的采集端[brave](https://github.com/openzipkin/brave)提供apm插件，
为concrete的服务执行、模块间调用进行性能跟踪。

zipkin搭建就不多说了，按照 https://github.com/openzipkin/zipkin#zipkin 的说明安装即可。

我的实验环境发布在 http://localhost:9411

## 开始使用

release模块中，增加`concrete-apm-zipkin`的依赖

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-apm-zipkin</artifactId>
        </dependency>
```

concrete.properties中配置
```properties
zipkin.location=http://localhost:9411
module.name=demo
```

module.name用来声明本模块的标签，一会在zipkin里可以看到

好了，把服务跑起来，各种访问。然后访问 http://localhost:9411 `Find Traces`

## Client效果

为了演示效果，我们在实现中做点罗嗦的，在GirlServiceImpl里，save的时候，并行调用远端10次加法，delete的时候，并行调用远端做5次加法。

impl模块，增加依赖
```java
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-client</artifactId>
        </dependency>
```

GirlServiceImpl，建一个线程池来做并行
```java
private final ExecutorService executorService = ExecutorsHelper.newFixedThreadPool(5);
```

注入一个DemoService的远端调用实例
```java
    @Inject
    @ConcreteClient("demoClient")
    private DemoService demoService;
```

修改saveGirl方法
```java
    @Override
    public void saveGirl(Girl girl) {
        IF.is(girlsMap.containsKey(girl.getName()), "Girl exists.");
        girlsMap.put(girl.getName(), girl);
        girlComingTopic.publish(girl);

        // step 3.7.1
        final NewGirlComing newGirl = new NewGirlComing();
        newGirl.setGirl(girl);
        scheduledExecutorService.schedule(new Runnable() {
            @Override
            public void run() {
                newGirlComing.publish(newGirl);
            }
        }, 5, TimeUnit.SECONDS);

        // step 3.9
        List<Runnable> runnables = new ArrayList<Runnable>();
        for(int i = 0; i < 10; i ++){
            final int finalI = i;
            runnables.add(new Runnable() {
                @Override
                public void run() {
                    demoService.add(finalI,finalI);
                }
            });
        }

        APM.parallel(executorService, runnables.toArray(new Runnable[0]));
    }
```

修改deleteGirl方法
```java
    @Override
    public void deleteByName(String name) {
        // step 3.6
        if (girlsMap.containsKey(name)) {
            Girl girl = girlsMap.get(name);
            girlsMap.remove(name);
            girlGoneTopic.publish(girl);
        }

        // step 3.9
        // 串行示例
        demoService.add(6,6);

        List<Runnable> runnables = new ArrayList<Runnable>();
        for(int i = 0; i < 5; i ++){
            final int finalI = i;
            runnables.add(new Runnable() {
                @Override
                public void run() {
                    demoService.add(finalI,finalI);
                }
            });
        }
        APM.parallel(executorService, runnables.toArray(new Runnable[0]));
    }
```

release模块增加jaxrs-client的依赖
```java
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs-client</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
        </dependency>
```

增加demoClient的描述`client.demoClient.properties`
```properties
location=http://localhost:8080/jaxrs
```

再跑一下，然后到zipkin里看看跟上次的有什么不同？

当前代码为：https://github.com/coodex2016/concrete-demo/tree/step3_8