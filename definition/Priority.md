# @Priority

定义服务原子的优先级，优先级越高的，意味着越优先获取系统资源来提供服务。如果定义在服务模块上，则说明此服务模块的所有服务原子默认使用该优先级。
例如：

```java
 public interface 为官之道 extends ConcreteService{   
    
    @Priority(3)
    void 做好事情();

    @Priority(10)
    void 不出错();
    
}

// or

@MicroService("自行车被盗")
@Abstract
public interface A extends ConcreteService{
    自行车 找回();
}
    
@MicroService("河源启一郎")
@Priority(10)
public interface B extends A{}

@MicroService("武汉市民")
@Priority(-1) //超出值域范围会自动调整到值域范围内
public interface C extends A{}
```

在Concrete提供的工具链中，jaxrs1.0和2.0的实现方式有所区别。

1.0是伪实现，因为使用servlet容器的线程池，只是在执行服务原子前，由工具链对其线程优先级进行了调整，但是该服务原子何时被执行是未知。

Jaxrs2.0规范中定义了异步的支持，所以jaxrs2.0的使用工具链提供的线程池，保障了高优先级的任务一定优先执行。