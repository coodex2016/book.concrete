# @AccessAllow

作用于服务原子上，调用声明服务原子需要什么角色。
未定义`@AccessAllow`的表明无需验证访问者身份
使用默认值的`@AccessAllow`表明系统的所用用户都可访问

例如：妇女或小孩可以使用绿色通道
```java
    @AccessAllow({"women", "children"})
    String 绿色通道();
```

如果在具体服务模块上定义了~~@RoleOwner~~`@Domain`，则说明需要指定~~属主~~领域下的角色。

例如：中国.官员 可以先走

```java

@MicroService("克拉玛依")
// RoleOwner("中国")
@Domain("中国")
public interface 火灾 extends ConcreteService{
    @AccessAllow({"官员"})
    String 先走();
}
```