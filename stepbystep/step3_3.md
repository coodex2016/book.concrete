# 3.3 签名验签

很多情况下，我们的系统需要对外提供数据对接接口服务，安全环境中还好说，如果放到公众环境中，不加限制的话，很容易被攻击，产生危害系统的数据，concrete针对这种情况，提供了签名验签的规范。

## @Signable

`@Signable`可用来装饰接口方法，也可用来装饰服务接口类。声明在服务接口类上时，则说明该服务接口类的所有接口方法（包含继承的）都需要签名验签。

启用签名验签功能需要注册上`SignatureInterceptor`，服务端好说，直接注册就行了。很多情况下，Client端并不一定有DI容器，
所以，需要根据情况注册，有DI容器的情况，在DI容器中注册，
没有DI容器的情况，使用ServiceLoader规范（META-INF/services/org.coodex.concrete.core.intercept.ConcreteInterceptor）注册。

## 签名验签要素

签名验签本质上有五个要素：内容、算法、干扰、客户key、签名

### 内容

需要签验的内容，concrete默认采用类OAuth的内容序列化方式，也可以自行扩展`org.coodex.concrete.core.signature.SignatureSerializer`

内容包含干扰、算法和客户key

### 算法

算法，默认支持`哈希算法withRSA`和`Hmac哈希算法`，也可以自行扩展`org.coodex.concrete.core.signature.IronPenFactory`实现其他的签验方案

### 干扰

为避免报文重试，我们在内容中定义干扰项，验签时可以对干扰项进行判定，比如同一个keyId，1分钟以内不能出现相同干扰项，或者当干扰项为时间戳是，误差不得大于5分钟等。

干扰项提供了两个可扩展的接口

- `org.coodex.concrete.core.signature.NoiseGenerator`，扩展如何生成干扰项，默认是随机整数
- `org.coodex.concrete.core.signature.NoiseValidator`，扩展如何检验干扰项，默认不检验

### 客户key

说明是谁签的，有一定的业务含义

### 签名

按照算法，使用客户key对应的密钥，对内容进行签名，并将签名按Base64编码

## KeyStore

签名验签就是基于密钥的算法，对于服务端而言，需要管理所有可接入者的对称密钥或者非对称公钥，concrete默认的HmacKeyStore是基于资源文件的，RSAKeyStore是基于资源路径`rsaKeys`的。
如需其他的管理方式（比如和业务数据挂钩），可自行扩展`org.coodex.concrete.core.signature.HmacKeyStore`和`org.coodex.concrete.core.signature.RSAKeyStore`

### 关键要素名重载

signature.properties是concrete约定重载Signable相关属性的属性文件

例如
```properties
property.sign=signature
property.keyId=appId
property.noise=disturb
property.algorithm=al
```

## 开始演示

说了一大通，直接用代码来体验吧还是。

大致思路如下
- 新建一个需要签名的接口服务，做一个空实现
- 服务端配上拦截器，配置好默认的签验信息
- 新建一个调用端，注册拦截器，配置各自的签验信息

### 定义接口，并实现

`org.coodex.concrete.demo.api.SignableService`
```java
package org.coodex.concrete.demo.api;

import org.coodex.concrete.api.ConcreteService;
import org.coodex.concrete.api.MicroService;
import org.coodex.concrete.api.Signable;
import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.concrete.demo.api.pojo.VehiclePlate;
import org.coodex.util.Parameter;

@Signable
@MicroService
public interface SignableService extends ConcreteService {

    String example(
            @Parameter("girl") Girl girl,
            @Parameter("plate")VehiclePlate vehiclePlate);
}
```

实现`org.coodex.concrete.demo.impl.SignableServiceImpl`

```java
package org.coodex.concrete.demo.impl;

import org.coodex.concrete.demo.api.SignableService;
import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.concrete.demo.api.pojo.VehiclePlate;

import javax.inject.Named;

@Named
public class SignableServiceImpl implements SignableService {

    @Override
    public String example(Girl girl, VehiclePlate vehiclePlate) {
        return String.format("%s has a car: %s", girl.getName(), vehiclePlate.getCode());
    }
}
```

### 发布服务的配置

`DemoBoot`注册签名验签拦截器

```java
    @Bean
    public SignatureInterceptor signatureInterceptor(){
        return new SignatureInterceptor();
    }
```

配置一个`signature.properties`
```properties
hmacKey.Lele=Davidoff is handsome
hmacKey.Feifei=Yes, that's right
hmacKey.Nana=I agree too
```

### 调用端

0.2.3快照版客户端签名开始支持模块化，我们模拟一个客户端调用多个服务端的模式来演示，先准备3个模块的配置，并且使用不同的签名算法。

signature.Lele.properties
```properties
keyId=Lele
algorithm=HmacMD5
hmacKey=Davidoff is handsome
```

signature.Feifei.properties
```properties
keyId=Feifei
algorithm=HmacSHA1
hmacKey=Yes, that's right
```
 
signature.Nana.properties
```properties
keyId=Nana
algorithm=HmacSHA256
hmacKey=I agree too
```

配置各模块的location
client.properties
```properties
Lele.location=http://localhost:8080/jaxrs
Feifei.location=http://localhost:8080/jaxrs
Nana.location=http://localhost:8080/jaxrs
Davidoff.location=alias:Lele
```

> #### Hint::
>
> 注意最后一个，说明Davidoff这个模块只是Lele这个模块的一个别名，也就是说，这两个模块是完全一致的，一会可以看一下结果

注册拦截器
`META-INF/services/org.coodex.concrete.core.intercept.ConcreteInterceptor`
```
org.coodex.concrete.core.intercept.SignatureInterceptor
``` 

`org.coodex.concrete.demo.invoker.SignatureExample`
```java
package org.coodex.concrete.demo.invoker;

import org.coodex.concrete.Client;
import org.coodex.concrete.demo.api.SignableService;
import org.coodex.concrete.demo.api.pojo.Girl;
import org.coodex.concrete.demo.api.pojo.VehiclePlate;
import org.coodex.util.Common;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SignatureExample {

    private final static Logger log = LoggerFactory.getLogger(SignatureExample.class);

    private static void invoke(String girlName, String carCode) {
        SignableService signableService = Client.getInstance(SignableService.class, girlName);
        Girl girl = new Girl(girlName);
        VehiclePlate vehiclePlate = new VehiclePlate();
        vehiclePlate.setCode(carCode);
        if (!Common.isBlank(carCode))
            vehiclePlate.setColor(2);
        log.info(signableService.example(girl, vehiclePlate));
    }

    public static void main(String[] args) {
        invoke("Lele", "中A8*228");
        invoke("Feifei", "国APX215");
        invoke("Nana", "牛B74110");
        invoke("Davidoff", null);
    }
}
```

跑起来看看控制台，签名相关信息会输出在控制台上。

注意最后一段
```
url: http://localhost:8080/jaxrs/OrgCoodexConcreteDemoApiSignableService/example
method: POST
headers:
	noise: 1551223104
	sign: I1hY7vW+UlZbuYWZuO3OTw==
	keyId: Lele
	X-CLIENT-PROVIDER: cocnrete-jaxrs-client 0.2.3-SNAPSHOT
	algorithm: HmacMD5
content:
{"plate":{},"girl":{"name":"Davidoff","stars":0}}
```
我们调用Davidoff模块时，使用的时Lele的keyId，也就是说，Davidoff就是Lele模块的别名。

大家可以自行改改看看结果。

截止到目前的代码： https://github.com/coodex2016/concrete-demo/tree/step3_3