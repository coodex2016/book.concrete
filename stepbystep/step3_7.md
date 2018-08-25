# 3.7 基于Token的消息推送

首先，Token是啥？Token是由后端发放给前端代表身份的令牌，所以，这个玩意肯定是用在前后端之间。

一样，我们先设想个场景，还是用GirlService，客户端订阅所有girl到达的消息，当新建一个girl的时候，延迟5秒向客户端推送girl到达信息;

好，动手。

## 准备

我们在api的pojo中增加一个新的类型，用来包含Girl和Girl的新建时间。

NewGirlComing
```java
package org.coodex.concrete.demo.api.pojo;

import org.coodex.concrete.message.MessageSubject;
import org.coodex.util.Common;

import java.io.Serializable;

@MessageSubject("newGirlComing")
public class NewGirlComing implements Serializable {
    private Girl girl;
    private String arrived = Common.now();

    public Girl getGirl() {
        return girl;
    }

    public void setGirl(Girl girl) {
        this.girl = girl;
    }

    public String getArrived() {
        return arrived;
    }

    public void setArrived(String arrived) {
        this.arrived = arrived;
    }
}
```

> #### Hint::
>
> 因为同一个客户端有可能需要订阅多种类型的数据，客户端需要根据主题来区分应该如何响应，
> 所以我们使用`MessageSubject`注解为这个消息声明一个Subject，
> 也可以让POJO实现`org.coodex.concrete.message.Subject`


## 服务端改造

GirlService新增订阅全部消息的接口

```java
    // step 3.7.1
    void subscribe();
```

GirlServiceImpl实现
```java
    // step 3.7.1
    @Queue("girlComing")
    private TokenBasedTopic<NewGirlComing> newGirlComing;

    @Inject
    private Token token;

    private final ScheduledExecutorService scheduledExecutorService =
            ExecutorsHelper.newSingleThreadScheduledExecutor();

```

```java
    private static final String key_all_girls = "all girl message subscribed";
    
    private boolean isSubscribed(){
        Boolean b = token.getAttribute(key_all_girls, Boolean.class);
        return b != null && b;
    }
    @Override
    public void subscribe() {
        if(!isSubscribed()) {
            synchronized (this) {
                if(!isSubscribed()) {
                    token.setAttribute(key_all_girls, Boolean.valueOf(true));
                    
                    newGirlComing.subscribe(new MessageFilter<NewGirlComing>() {
                        @Override
                        public boolean handle(NewGirlComing message) {
                            return true;
                        }
                    });
                }
            }
        }
    }
```

> #### Hint::
>
> TokenBased，那么必然需要Token，在concrete里，Token需要被明确使用以后才会把令牌号发放给客户端，所以，我们用了一个setAttribute来激活Token
>
> concrete服务运行在多线程环境，所以，我们用双重校验锁的机制来进行订阅

`saveGirl`方法实现修改
```java
        // step 3.7.1
        final NewGirlComing newGirl = new NewGirlComing();
        newGirl.setGirl(girl);
        scheduledExecutorService.schedule(new Runnable() {
            @Override
            public void run() {
                newGirlComing.publish(newGirl);
            }
        }, 5, TimeUnit.SECONDS);
```
好了，服务端修改完成。

## 调用端


```html
<button id="subscribe">subscribe</button>
<div id="content"></div>
```

```javascript
            concrete.configure({
                root: '/jaxrs',
                onBroadcast: function (msgId, host, subject, data) {

                    // 3.7.1
                    var el = '<div>'subject + ': ' +data.arrived + ' ' + data.girl.name +'</div>';
                    $('#content').append($(el));
                }
            });

            var girlService = concrete.module("GirlService");
            
            // ......
            
            $('#subscribe').click(function () {
                girlService.subscribe().then(function (value) {
                    concrete.polling();
                });
            })
```


## 生成包含Polling的脚本

api-helper模块

注意，jaxrs模式下，concrete.polling()依赖于`Polling`服务，默认的api生成不会带它，所以，我们需要把它包含进来

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>concrete-jaxrs</artifactId>
        </dependency>
```

```java
        //生成jQuery调用服务的代码
        API.generateFor("demo.jquery",
                // 3.7.1
                Polling.class.getPackage().getName(),
                DemoService.class.getPackage().getName());
```

拷贝到发布模块的`resources/static/jquery`目录下

> #### Hint::
>
> 建议你的api_gen.demo.jquery.properties的path直接配置到这个目录，能省很多事

好了，DemoBoot跑起来，访问 http://localhost:8080/jquery/index.html 
点击subscribe按钮

用GirlServiceExample来发送girls信息，等几秒，看

场景达到了，当前代码为：https://github.com/coodex2016/concrete-demo/tree/step3_7_1

## 使用订阅切片方法

我们改动以下几点：

新建`**.impl.interceptor.AllGirlSubscribeInterceptor`，把实现中的代码移植过来。
```java
package org.coodex.concrete.demo.impl.interceptor;

import org.coodex.concrete.common.Token;
import org.coodex.concrete.core.intercept.AbstractTokenBasedTopicSubscribeInterceptor;
import org.coodex.concrete.demo.api.pojo.NewGirlComing;
import org.coodex.concrete.message.MessageConsumer;
import org.coodex.concrete.message.MessageFilter;

import javax.inject.Inject;
import javax.inject.Named;

@Named
@MessageConsumer(queue = "girlComing")
public class AllGirlSubscribeInterceptor
        extends AbstractTokenBasedTopicSubscribeInterceptor<NewGirlComing> {

    private static final String key_all_girls = "all girl message subscribed";
    @Inject
    private Token token;

    private boolean isSubscribed() {
        Boolean b = token.getAttribute(key_all_girls, Boolean.class);
        return b != null && b;
    }

    @Override
    protected MessageFilter<NewGirlComing> subscribe() {
        token.setAttribute(key_all_girls, Boolean.valueOf(true));

        return new MessageFilter<NewGirlComing>() {
            @Override
            public boolean handle(NewGirlComing message) {
                return true;
            }
        };
    }

    @Override
    protected boolean check() {
        return !isSubscribed();
    }
    
    @Override
    protected boolean checkAccountCredible() {
        return false;
    }
}
```


修改实现
```java
    @Override
    public void subscribe() {
        // 3.7.2
        token.setAttribute("a", "a");
    }
```

好，同样的，看看效果。

差异在哪？

- 方案一需要自行维护订阅，当不再需要订阅时，需要手动cancel；方案二不需要，当令牌失效时，切片会及时取消已订阅的内容
- 方案一只能在特定接口进行订阅; 方案二在所有接口都可以触发

> #### Hint::
>
> 有几点需要说一下：
> - 实现里为什么要`token.setAttribute("a", "a");`？ 因为token只有被使用过才会把令牌传递给客户端，客户端拿不到令牌就没法轮询，
> 而我们一般的业务场景都会根据令牌中当前用户信息进行订阅。所以，这里纯粹是为了演示。
> - 切片中，我们重载了`checkAccountCredible`方法，和上面一样，切片默认需要当前账户有效并可信，而我们演示场景没有账户信息，
> 所以改变了抽象类的行为。
>
> 以上这两点都是为了演示用的，大家根据实际情况进行选择。
>
> 再有就是，控制好订阅的状态，不要出现同一个token重复订阅

当前代码为：https://github.com/coodex2016/concrete-demo/tree/step3_7_2