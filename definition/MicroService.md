# @MicroService

可注解在接口类型上和方法上。用来声明这是一个服务模块或服务原子。`value`设定模块或原子的名称。

## 注解在接口类型上

注解在接口类型上时，则声明该接口类型是一个具体的对外服务模块，如果该模块并不是一个具体的服务模块，需要配合`@Abstract`注解。
默认名称为该接口的java类全名，如注解了`@Abstract`，则默认名称为java类名。

`@Abstract`的作用是定义**服务模块**的继承链，用以描述模块之间的继承关系。该继承关系与java继承概念类似但不等同，例如:

    接口继承关系为：
    ConcreteService 
    <-- 生物 
    <-- @MicroService("动物") @Abstract Animal 
    <-- 脊椎动物 
    <-- @MicroService("鱼类") @Abstract Fish 
    <-- @MicroService("鲤鱼") Carp
    
    服务模块继承关系
    动物 <-- 鱼类 <-- 鲤鱼

最终对外提供鲤鱼服务。各服务提供者应该根据服务继承关系提供相应的表现形式。
~~在Concrete自带工具链jax-rs支持中，葱油鲤鱼服务的资源位置为:/动物/鱼类/鲤鱼/葱油/_{哪一条}_~~



## 注解在方法上

因为ConcreteService的所有方法默认均为服务原子，因此，@MicroService的作用更多用于更改方法的服务名。
~~例如：
在Concrete自带工具链jax-rs支持中，默认new为PUT方法的谓词，而new是java的关键字，无法定义new()的接口，可以如下定义：~~
```java
    // 已失效
    // @MicroService("new")
    // POJOType newObject();
```
