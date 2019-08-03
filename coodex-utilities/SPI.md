# SPI - Service Provider Interfaces

java 1.6起增加了SPI机制，定义好一个Provider Interface，然后在`META-INF/services`目录下建一个同名的文档，按行添加该接口的实现类，即可通过ServiceLoader获取所有的Provider实例，是OOAD设计原则中依赖反转思想的体现，让系统可扩展性更好。

java原生的SPI机制只能由jvm根据类来提供实例，按DI的理念来讲，我们不应该关注或者限定实例的提供方式，所以`coodex`设计了一个既兼容java SPI且扩展性更强的SPI机制。因为宝宝对迭代器理解有限，所以coodex SPI并没有像Java SPI那样提供迭代模式。

## org.coodex.util.ServiceLoader&lt;T>

### Map<String, T> getAll()

获取所有此类型的实例，coodex SPI为实例增加了一个名称的属性，参考了`javax.inject`规范的name设计，在扩展和服务选择时可以更方便的操作

### T get(Class<? extends T>)

根据一个类型获取服务实例

### T get(String)

根据名称获取服务实例

### T get()

获取实例，当存在多个服务实例时，可通过重载`ServiceLoaderImpl`的conflict系列方法来解决

### T getDefault()

获取默认的服务实例。coodex SPI建议每个Loader都能提供一个默认的服务实例，减少使用复杂度

## org.coodex.util.ServiceLoaderImpl&lt;T>

coodex SPI的实现

### usage

例如，我们有一个interface A

```java
private static final ServiceLoader<A> A_SERVICE_LOADER = new ServiceLoaderImpl<A>(){};
```

## 扩展点

### org.coodex.util.ServiceLoaderProvider

coodex SPI机制相对java SPI来讲，支持多种ServiceLoader，需要增加你的Loader机制时，只需要实现一个ServiceLoaderProvider即可

已有的实现包括

#### org.coodex.util.JavaUtilServiceLoaderProvider

coodex-utitlies中，基于java.util.ServiceLoader的Provider

#### org.coodex.concrete.common.ConcreteServiceLoaderProvider

concrete-core中，基于concrete BeanProvider的ServiceLoaderProvider