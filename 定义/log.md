# 操作日志

concrete 提供了两个注解定义操作日志的行为。

## OperationLog

| 属性 | 说明 |
| --- | --- |
| category | 操作日志分类 |
| formatterClass | 如何格式化，concrete工具链提供默认，推荐使用FreemarkerMessageFormatter |
| patternLoaderClass | 模版如何获取，同ErrorCode，键值定义为category.subClass |
| loggerClass | 如何持久化日志，优先级低于LogAtomic的loggerClass |

## LogAtomic

| 属性 | 说明 |
| --- | --- |
| subClass | 操作日志子类 |
| loggerClass | 如何持久化日志，非默认值时可覆盖OperationLog的loggerClass |

## 使用

### in spring

```xml
<bean class="org.coodex.concrete.spring.aspects.OperationLogAspect"></bean>
```

### 定义
```java
@MicroService()
@OperationLog(category = "test")
public interface ServiceExample extends ConcreteService{
    
    @LogAtomic(subClass="test") // 使用模版为test.test
    void logTest();
}
```

### 实现

```java
    @Override
    public void logTest(){
        // org.coodex.concrete.common.ConcreteContext.LOGGING
        LOGGING.get().put("logTest", "ok");
    }

```

如上所示，当logTest成功执行后，会按照定义进行日志渲染和持久化。concrete工具链默认使用Slf4J的info级别持久化日志。

### 扩展

实现接口: org.coodex.concrete.common.OperationLogger, 对操作日志进行持久化。