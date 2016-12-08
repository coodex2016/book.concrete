# 服务原子切片

在Spring中使用时
```xml
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

Concrete工具链提供了3种切片来支持Concrete的定义。

## RBAC

```xml
    <bean class="cc.coodex.concrete.spring.aspects.RBAC_Aspect"></bean>
```

## Bean Validation

```xml
    <bean class="cc.coodex.concrete.spring.aspects.BeanValidationAspect"></bean>
```
可以通过实现ViolationsFormatter来修改默认的信息格式化。

## ServiceTiming

```xml
    <bean class="cc.coodex.concrete.spring.aspects.ServiceTimingAspect"></bean>
```
在`serviceTiming.properties`里定义验证规则，规则模版如下:

    ruleName.type = ... 
    ruleName.properties = some value
    ...
    
    type可以为以下值:
       timerange: 根据时间提供服务
            properties：
                range: HH:mm-HH:mm;..
                     
       workday: 根据日期提供服务
            properties:
               weekday: 每周哪几天是工作日，0-6分别代表周日-周六，默认为1,2,3,4,5
               restDay: 哪些天是休息日，格式yyyy-MM-dd，多个休息日使用“,”分隔，属性名中D大写
               workday: 哪些天是工作日，格式yyyy-MM-dd，多个工作日使用“,”分隔
            验证优先级
               in workday: true
               in restDay: false
               return is weekday
               
       自己实现ServiceTimingChecker的类名
       自定义class时：
           1、属性名与class中的非static非final的String类型的域相对应，自动装载
           2、自定义的class需要能够无参数构造实例

如[@ServiceTiming](../定义/ServiceTiming.md)中的示例
```java
    @ServiceTimming({"工作日","9_11或14_16"})
    Paper 盖章();
```

`serviceTiming.properties` 配置应该如下：
    
    工作日.type = workday
    工作日.weekday = 1, 2, 3, 4, 5
    工作日.restDay = 2016-01-01 ... 节假日配置
    
    9_11或14_16.type = timerange
    9_11或14_16.range = 09:00-11:00; 14:00-16:00
    
该配置支持动态调整，有节假日或临时调整时，提前修改配置即可生效。比如说，2016年12月7日8日调休，10日11日上班，那么在7号之前设置上

    工作日.restDay = 2016-12-07, 2016-12-08
    工作日.workday = 2016-12-10, 2016-12-11

即可，无须重新启动
    

## 切片链方式

```xml
    <bean class="cc.coodex.concrete.spring.aspects.ConcreteAOPChain">
        <constructor-arg index="0">
            <list>
                <bean class="cc.coodex.concrete.core.intercept.BeanValidationInterceptor"></bean>
                <bean class="cc.coodex.concrete.core.intercept.RBACInterceptor"></bean>
                <bean class="cc.coodex.concrete.core.intercept.ServiceTimingInterceptor"></bean>
            </list>
        </constructor-arg>
    </bean>
```
