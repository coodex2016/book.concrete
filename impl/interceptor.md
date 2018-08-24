# 拦截器

| 拦截器 | 服务端 | 客户端 | 本地调用 | 调试环境 | 非concrete环境 |
| ----- | ----- | ----- | ------- | ------ | ------------ |
| RBACInterceptor | true | - | - | true | true |
| SignatureInterceptor | true | true | - | - | - |
| OperationLogInterceptor | true | - | true | true | - |
| BeanValidationInterceptor | true | - | true | true | - |
| MaximumConcurrencyInterceptor | true | - | true | - | - |
| ServiceTimingInterceptor | true | - | true | true | - |
| AbstractTokenBaseTopicSubscribeInterceptor | true | - | - | true | - |

> #### Hint::关于客户端拦截器注册
>
> Client端并不一定有DI容器，所以，需要根据情况注册，有DI容器的情况，在DI容器中注册，
> 没有DI容器的情况，使用ServiceLoader规范（META-INF/services/org.coodex.concrete.core.intercept.ConcreteInterceptor）注册。
> 当然，两种都注册上也可以，会重复执行而已。

