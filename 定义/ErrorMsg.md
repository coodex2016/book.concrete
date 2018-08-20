# @ErrorMsg

@ErrorMsg用于定义各种错误号的表现形式，可注解于Type或具体错误号定义上。

* Class<? extends MessagePatternLoader> patternLoaderClass
    指定使用哪个错误信息模版加载类。
    Concrete工具链提供了一个默认的加载类[ResourceBundlesMessagePatternLoader](../工具链/ResourceBundlesMessagePatternLoader.md)
    
* Class<? extends MessageFormatter> formatterClass
    指定使用哪个格式化类。
    Concrete工具链提供了一个默认的格式化类[JavaTextFormatMessageFormatter](../工具链/JavaTextFormatMessageFormatter.md)，
    推荐使用插件中提供的基于FreeMarker模版引擎的实现[FreemarkerMessageFormatter](../工具链/FreemarkerMessageFormatter.md)
    
* String value
    仅在注解在具体错误号定义上时有效。用来指明格式化模版或模版的键值。当以花括号括其时说明花括号内的内容为模版键值，将交由patternLoaderClass负责加载该模版，默认键值为message.具体错误号。
    

    
例如：
```java
    @ErrorMsg(value="${o1.姓名} 开着 ${o2.车牌} 把 ${o3.车牌} 给撞了！",
        formatterClass=FreemarkerMessageFormatter.class)
    public static final int 撞车 = 5000000;


    //服务原子的实现中如下使用
    IF.is(是否撞了, SomeErrorCodes.撞车, 司机, 车, 被撞的车);

```