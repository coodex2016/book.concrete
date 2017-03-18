# TokenManager

需要使用Token时，直接使用
```java
    Token token = TokenWrapper.getInstance();
```
或者把TokenWrapper配置成bean，自动装载

```xml
    <bean class="org.coodex.concrete.core.token.TokenWrapper"></bean>
```

```java
    @Inject Token token;
```

## 单节点

用于获得当前服务原子的令牌信息，Token里面可以保存可序列化的对象。


## 集群支持

当我们的服务发布于集群环境时，可以选用基于redis或memcached的TokenManager

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>shared-cache-jedis</artifactId>
        </dependency>
```

或者

```xml
        <dependency>
            <groupId>org.coodex</groupId>
            <artifactId>shared-cache-memcached</artifactId>
        </dependency>
```

```xml
    <bean class="org.coodex.concrete.core.token.sharedcache.SharedCacheTokenManager"/>
```

### concrete.properties配置

    # 令牌缓存的类型，可选xmemcached或jedis
    tokenCacheType = xmemcached | jedis
    # 令牌最大空闲时间
    sharedCacheTokenManager.maxIdleTime = 60

### redis的配置文件

sharedcache-jedis.properties

    redisServers = server1, server2 ....
    pool.minIdle = 
    pool.maxIdle =
    pool.maxTotal = 
    defaultMaxCacheTime = 3600
    
### memcached配置

sharedcache-xmemcached.properties

    memcachedServers = server1, server2 ...
    poolSize = 1
    user._server1_ =
    pwd._server1_ =
    ....