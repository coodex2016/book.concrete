# PojoMocker

    模拟Pojo

## API

### MockerFacade 主要接口

```java

public static <T> T mock(GenericType<T> genericType);

public static <T> T mock(GenericType<T> genericType, Class context);

public static <T> T mock(Type type);

public static <T> T mock(Type type, Class... context);

public static <T> T mock(Method method);

public static <T> T mock(final Method method, Class ... context);

```

### 主要注解

- @COLLECTION，定义数组、Collection类的大小
    - int[] size()， 数组，表示数组(Collection视作数组)各维度的大小，为-1则表示随机
- @MAP，定义MAP模拟属性
    - Class keyType()，未指定键值类型时可通过keyType定义
    - Class valueType()，未指定值类型时可通过valueType定义
    - Class keyMocker()，指定键使用什么样的方式模拟，使用该属性的此类型注
    - Class valueMocker()，指定值使用什么方式模拟，使用该属性的此类型注解
    - int size(), map模拟大小
- @Deep，定义相同类型的模拟深度
    - int min(), 最小深度
    - int max(), 最大深度
- @Relation，定义模拟关联，例如，性别属性关联自身份证号，则可以在性别属性上定义 `@Relation(properties = "身份证号")`
    - String[] properties()， 此属性所关联的属性集。当前版本仅支持同一POJO的属性
    - Class<? extends RelationPolicy> policy()，怎么从关联的属性值中获得此属性的值
    
### @Mock

使用Mock对扩展的Annotation进行注解，实现一个Mocker<YourAnnotation>服务并注册，即可。

例如

```java
/**
 * 模拟产生id,支持Long/long/String
 * Created by davidoff shen on 2017-05-15.
 */
@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Mock
public @interface ID {
}
```

```java
public class IdCardMocker extends AbstractMocker<IdCard> {
    // ......
}
```

## 已有Mock注解

### @BYTE, @SHORT, @INTEGER, @LONG, @FLOAT, @DOUBLE

可指定最大值最小值

### @BOOLEAN

50%几率随机true/false

### @CHAR

可定义字符范围，可通过mock.properties的default.chars.range设定默认范围

### @STRING

定义字符串的字符范围，可通过mock.properties的default.string.range设定默认范围

### concrete-api @ID

模拟id，支持String/Long/long

### concrete-api @Name

模拟中文姓名

### concrete-api @IdCard

模拟身份证号，可设定最大最小年龄（默认最小5，最大90）、性别（男，女，随机）、行政区划过滤、规格（15，18，随机）

### concrete-api @DateTime

模拟时间，可设置最大最小日期范围

### concrete-api @VehicleNum

模拟车牌号，可设置是否挂车、是否教练车、省份范围

### concrete-api @MobilePhoneNum

模拟手机号，appleStyle 为 true则返回 xxx-xxxx-xxxx

### concrete-api @EMail

模拟电子邮件地址，可设置domains，仅在domains中模拟

