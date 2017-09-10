# 签名验签

- 签名必须包含noise和sign可选algorithm/keyId，可通过`signature.properties`配置 `property.***`
- noise/sign/algorithm/keyId均可显示定义在单元参数(不推荐)中，如不定义则通过Subjoin传递
- 如果返回值为Signature类型，则对返回值进行签名
- 需要实现SignatureSerializer，将所有数据按照规则拼合，客户端与服务端需要维持一致，concrete-core提供默认规则
- 需要实现IronPenFactory, concrete-core提供基于Hmac/RSA实现，可扩展各自的key获取实现

```java
    @Signable()
    List<Book> findByAuthorLike(String author);

    @Signable(algorithm = "HmacSHA1")
    List<Book> findByPriceLessThen(int price);
```

Signable属性
paperName 默认为空，对签名进行分类，通过`signature.properties`来定义具体的签名方式
    - 配置优先级`algorithm.paperName` > `algorithm`: 签名算法
    - `keyId.paperName` > `keyId` : 客户端keyId
algorithm 签名算法，默认SHA256withRSA，优先级低于`signature.properties`的配置
serializer 序列化对象，默认使用concrete工具链提供的序列化算法(按键字典序 键=值模式拼接 urlEncode)

## 工具链

    工具链提供了默认的支持

### 关于signature.properties

    signature.properties是工具链中约定重载Signable相关属性的属性文件

- 重载algorithm等键值

因各系统对算法、签名、干扰、客户key的定义不一样，可以通过signature.properties进行重载

例如：

```properties
property.sign=signature
property.keyId=appId
property.noise=disturb
property.algorithm=al
```

### 算法支持

    默认支持支持HmacXXX/XXXwithRSA，默认SHA256withRSA

可自行重载`org.coodex.concrete.core.signature.HmacKeyStore`/`org.coodex.concrete.core.signature.RSAKeyStore`
默认的KeyStore规则如下

#### HmacKeyStore 默认实现

`signature.properties`

key明文配置，配置优先级：
`hmacKey.paperName.keyId` >
`hmacKey.paperName` >
`hmacKey.keyId` >
`hmacKey`


#### RSAKeyStore 默认实现

`signature.properties`

私钥配置使用base64编码，优先级
`rsa.privateKey.paperName.keyId` >
`rsa.privateKey.paperName` >
`rsa.privateKey.keyId` >
`rsa.privateKey`
resource:
`paperName.keyId.pem` >
`paperName.pem`

公钥配置使用base64编码,优先级
`rsa.publicKey.paperName.keyId` >
`rsa.publicKey.paperName` >
`rsa.publicKey.keyId` >
`rsa.publicKey`
resource:
`paperName.keyId.crt` >
`paperName.crt`


服务端建议自行实现RSAKeyStore

### Interceptor


`org.coodex.concrete.core.intercept.JaxRSSignatureInterceptor`【已废弃】

`org.coodex.concrete.core.intercept.SignatureInterceptor`