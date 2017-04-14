# Concrete 

Concrete 是一种基于java的服务定义规范。

https://github.com/coodex2016/concrete.coodex.org


## 与0.1.0的不同

差异还是蛮大的，基本上算是完全~~重构~~[^1]构建了一个版本，降低了工具链自身与spring的耦合度，去掉了默认的一些模块。

主要差别有

|  | 0.1.0 | 0.2.0 |
| --- | --- | --- |
| 服务提供 | 基于Jaxrs，使用POST方法 | 贴近RESTFul风格，支持服务原子优先级 |
| 客户端 | 基于sill，耦合了界面的一些元素，函数式调用 | 基于jQuery，对象式调用，Promise模式；提供java客户端 |
| 错误信息 | 无参数，前后端均需定义 | 支持模版，在设计中定义，根据客户端语言环境选择信息模版 |
| api手册 | 前端手册，基于asciidoctor | 前后端分别提供手册，基于Gitbook |
| 文档 | 无 | 这本就是 |

------

## 2017-04-14

- JaxRS支持：取消只有1个POJO的限制。文档、客户端工具链同步更新


## 2017-04-13

- 修复生成文档时，泛型服务的POJO类型继承问题

## 2017-04-12
- 增加Limiting注解，定义业务限流接口，保障系统的可用性
- 提供最大业务并发量模型的默认实现，后续扩展令牌桶、漏桶模型的默认实现
    - 策略配置文件 limiting.maximum.concurrency.properties
    ```properties
      # 默认策略的最大业务并发量
      max
      # 指定策略的最大业务并发量
      strategyName.max
    ```
- [api工具](工具链/API.md)增加angular支持，支持angular2+

## 2017-03-27

- 调整客户端访问机制，定义SSLContextFactory接口，提供部分实现
    - JSSEDefaultSSLContextFactory: 生产JSSE的默认SSLContext
    - AllTrustedSSLContextFactory：信任所有server端证书，不建议使用，中间人攻击数据泄漏的哗哗的
    - X509CertsSSLContextFactory：cocnrete默认实现，只信任指定资源目录下的证书
        ```properties
        # concrete.properties
        
        # 默认路径
        trusted.certs.path
        
        # 各domain的资源路径, domain全小写
        # trusted.certs.path.domain(.port), 例如：
        trusted.certs.path.example.coodex.org
        ```
        参考InsertCerts.java 提供工具方法
        ```java
        X509CertsSSLContextFactory.saveCertificateFromServer(String host, int port, String storePath)
        ```
        获取指定host port的证书
         


## 2017-03-25
- 作废CommonRepository接口。这是一个草率的设计，不推荐。对于大部分系统都会采用读写分离，应该从设计上就明确区分出A仓库和T仓库，A仓库不应该有CUD，T仓库不应该支持动态查询，防止程序误用。

## 2017-03-24

- concrete-core 增加ConcreteCache对象，用来解决需要定期失效的缓存数据模型，例如：当实时性要求不高时，用户的角色可以缓存下来，10分钟后再重新加载新的角色，减小数据获取的开销
```properties
# concrete.properties
# 线程池大小，根据具体应用的负载考虑，默认为1
cache.thread.pool.size=1
# 缓存对象生命周期，默认10分钟。也可以通过构造函数自行传入单位和时限
cache.object.life=10
```



## 2017-03-22

- 增加[SaaS](工具链/SaaS.md)支持


## 2017-03-21

- 增加分页相关的POJO抽象定义
- 增加PageCopier通用方法，从spring data的page对象复制成PageResult
- 修改ConcreteException，当最后一个参数为Throwable时，将其设置为引发ConcreteException的Cause

## 2017-03-18
- 315搞到了coodex.org域名，撒花
- 重构包名 -> org.coodex
- 重构groupId -> org.coodex
- Apache Licence 2.0
- git 迁移到 https://github.com/coodex2016/concrete.coodex.org
- 基于 [V5哥](mailto:sujiwu@126.com) 的设计思路，添加jpa和spring data jpa的通用模块，定义了PO/VO转换的规约
- 318 sonatype Staging repositories have been prepared for org.coodex，发布0.2.0
- 文档域名也迁移到coodex.org
- 开启0.2.1-SNAPSHOT，继续功能补充

## 2017-03-09
- 修复BigString注解的全部缺陷
- JavaClient模块强化，增加AOP功能
    - 继承org.coodex.concrete.core.intercept.AbstractInterceptor，重载有关方法
    - META-INF/services/org.coodex.concrete.core.intercept.ConcreteInterceptor 中增加需要使用到的拦截器
    - org.coodex.concrete.jaxrs.Client.getUnitFromContext：根据拦截器上下文获取Unit信息

## 2017-02-21

[@Description](定义/Description.md) 增强装饰功能。
可以装饰pojo属性和parameter。pojo属性指public field和getXXX isXXX(boolean)


## 2017-02-09

增加token Cookie path设定。原逻辑根据各模块的baseUri设置cookie path，主要考虑了同功能负载均衡可共享cookie，当多模块分离部署需共享cookie时，会无法获取。
    
    concrete.properties 增加配置项: jaxrs.token.cookie.path，默认为使用baseUri，否则使用设定值




## 2016-12-20

将账户是否有效的属性从Account接口中移至Token中。Account是客观账户实体，Token是账户认证后的状态。
Token中增加了当前账户相关的接口。


## 2016-12-15

增加[附件管理模块](工具链/fileServer.md)及基于Jax-RS 2.0 和 [FastDFS](https://github.com/happyfish100)的参考实现。

## 2016-12-10
调整Jaxrs的令牌模式，不再依赖servlet container环境。


## 2016-12-09

* 修改RoleOwner为Domain，为服务模块延伸了领域概念。

* 增加特权的定义
    * `*` 表示绝对特权，所有服务原子均有权访问
    * `*.role` 表示任意领域内的role角色
    * `domain.*` 表示领域内的特权，既该领域内所有服务均可使用

## 2016-12-07

* 工具链增加了[java客户端](工具链/JavaClient.md)

--------
[^1]: 重构一词使用有误。代码重构: [指对软件代码做任何更动以增加可读性或者简化结构而不影响输出结果](https://zh.wikipedia.org/wiki/%E4%BB%A3%E7%A0%81%E9%87%8D%E6%9E%84)，而实质上，concrete 0.2.0改变了较多的行为和结果

