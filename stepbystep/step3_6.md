# 3.6 发布订阅

之前提到过，concrete模块间的耦合提供两种支持，紧耦合通过[2.2 java端调用服务](step2_2.md)达到，松耦合则通过`发布订阅`达到。

我们来设计一个应用场景，改造GirlServiceImpl，当新增一个Girl的时候，有一个关注者来hi她一下，当移除一个Girl的时候，一个关注者来bye她一下。

ok，开始动手。

## 在GirlServiceImpl里注入两个不同的逻辑队列主题

发布订阅模式的主题必须是可序列化的，所以，先让Girl可序列化
```java
public class Girl implements Serializable{
    // 原有代码不动
}
```

GirlServiceImpl里注入两个发布订阅主题
```java
    // step 3.6
    @Queue("girlComing")
    private Topic<Girl> girlComingTopic;

    // step 3.6
    @Queue("girlGone")
    private Topic<Girl> girlGoneTopic;
```

> #### Hint::
>
> 发布订阅的一个约定是，相同的队列名、相同的Topic类型、相同的消息类型视为一个主题。
>
> 注意，这种方式并不符合DI规范，这是目前concrete-core-spring能力不足的原因，也希望大神们~~带我飞~~能帮我解决这个问题。

新来Girl的时候在girlComingTopic发布一个消息
```java
    @Override
    public void saveGirl(Girl girl) {
        IF.is(girlsMap.containsKey(girl.getName()), "Girl exists.");
        girlsMap.put(girl.getName(), girl);
        // step 3.6
        girlComingTopic.publish(girl);
    }
```

移除Girl的时候在girlGoneTopic发布一个消息
```java
    @Override
    public void deleteByName(String name) {
        // step 3.6
        if(girlsMap.containsKey(name)) {
            Girl girl = girlsMap.get(name);
            girlsMap.remove(name);
            girlGoneTopic.publish(girl);
        }
    }
```

好了，消息发布端就完成了。虽然还没有达到我们的预期目标，但是，消息生产者的逻辑已经完整了。之后这个主题被多少消费者关注与生产者无关，这就是松耦合了。

## 增加几个消费者

我们约定，把消息消费者放在**.impl.observer下

### Davidoff.java

```java
package org.coodex.concrete.demo.impl.observer;

import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.concrete.message.MessageConsumer;
import org.coodex.concrete.message.Observer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.inject.Named;

@Named
@MessageConsumer(queue = "girlComing")
public class Davdoff implements Observer<Girl> {
    private final static Logger log = LoggerFactory.getLogger(Davdoff.class);

    @Override
    public void update(Girl message) throws Throwable {
        if (message != null) {
            if (!"Davidoff".equals(message.getName())) {
                log.info("Hi, {}.", message.getName());
            } else {
                log.info("......");
            }
        }
    }
}
```

也就是每来一个girl，Davidoff会对她say hi.来一个跟自己同名的，无语。

我们先解析一下这个代码
- `@Named`，330规范，说明这个类应该被注册
- `@MessageConsumer(queue = "girlComing")`，说明这个类是girlComing消息队列的消费者
- `implements Observer<Girl>`，关注的消息是Girl

### Doubility

```java
package org.coodex.concrete.demo.impl.observer;

import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.concrete.message.MessageConsumer;
import org.coodex.concrete.message.Observer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.inject.Named;

@Named
@MessageConsumer(queue = "girlComing")
public class Doubility implements Observer<Girl> {
    private final static Logger log = LoggerFactory.getLogger(Doubility.class);

    @Override
    public void update(Girl message) throws Throwable {
        if (message != null) {
            if (!"Davidoff".equals(message.getName())) {
                log.info("Wow! {}, mua", message.getName());
            } else {
                log.info("You are handsome.");
            }
        }
    }
}
```

### Qiqi

```java
package org.coodex.concrete.demo.impl.observer;

import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.concrete.message.MessageConsumer;
import org.coodex.concrete.message.Observer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.inject.Named;

@Named
@MessageConsumer(queue = "girlGone")
public class Qiqi implements Observer<Girl> {
    private final static Logger log = LoggerFactory.getLogger(Qiqi.class);

    @Override
    public void update(Girl message) throws Throwable {
        if (message != null) {
            log.info("Oh no, I can not let you go, {}", message.getName());
        }
    }
}
```

好了，去掉DemoBoot里的模拟参数，去掉client-invoker的moduleMock.properties，把DemoBoot跑起来，然后用GirlServiceExample调用一下服务，看服务端控制台。

## 发布订阅的作用

### 剥离关注点

目前的效果，和观察者模式是一样一样的，那么concrete的发布订阅优势在哪呢？

优势在**剥离了对具体队列实现的关注**。开发者不用考虑是activeMQ还是rabbitMQ，
只要按照设计，消息生产者端在合适的时候使用合适的主题发布消息，消息消费者端只需要针对消息到达时按照需求、设计完成应该做的事情就行了。

### 易扩展

同样的，concrete发布订阅模型也是一个可扩展的。扩展点主要有两个，Topic和Courier，在此不细说了。


> #### Hint::
>
> concrete的队列定义是抽象的，具体用什么样的传输方式由queue.properties或queue.队列名.properties来指定，默认为local，即本地异步调用。
>
> 当我们的模块部署是跨虚拟机的情况下，我们需要使用具体的消息队列来传输。concrete提供activeMQ和rabbitMQ的实现，有兴趣的可以自己体验一下
> 
> #### activeMQ
>
> 依赖`concrete-courier-jms-activemq`，配置`queue.girlComing.properties` 和 `queue.girlGone.properties`
> ```properties
> destination=jms::activemq:your_activemq_url
> ```
>
> #### rabbitmq
>
> 依赖`concrete-courier-rabbitmq`，配置`queue.girlComing.properties` 和 `queue.girlGone.properties`
> ```properties
> destination=rabbitmq:amqp://username:password@host:port/virtualHost
> ```

截止到目前，代码版本为： https://github.com/coodex2016/concrete-demo/tree/step3_6