# Concrete定义规范

```java
@MicroService
public interface SomeService extends ConcreteService{
    
    @AccessAllow
    String someMethod(@Parameter("paramName") String paramName);
    
}
```

## 主要元素

| 名称 | 说明 |
| --- | --- |
| ConcreteService | 所有服务接口都需要继承自ConcreteService |
| [@MicroService](MicroService.md) | 可注解在接口类型上和方法上。用来声明这是一个服务模块或服务原子。value设定模块或原子的名称。 |
| @NotService | 对于接口中所有方法，默认是定义成服务原子，如某方法不想对外暴露服务接口，可使用@NotService注解在该方法上。当然，更合理的方案应该是把业务设计和技术设计分离开。 |
| [@Priority](Priority.md) | 定义服务执行的优先级。优先级的值域为\[1, 10\]，同Thread优先级 |
| [@AccessAllow](AccessAllow.md) | 定义访问指定服务原子时，需要具备什么角色 |
| [@ServiceTiming](ServiceTiming.md) | 定义服务原子可访问的时间段。 |
| [AbstractErrorCode](AbstractErrorCodes.md) | 定义服务的错误信息 |
| [@ErrorMsg](ErrorMsg.md) | 定义错误的表现形式 |
| [@Description](Description.md) | 用来定义服务模块或服务原子文档化时的信息 |
| [@Signable](Signable.md) | 签名验签 |


