# 基于Freemarker的MessageFormatter

o+index就是objects里的索引, 从1开始

例如：
```java
    @ErrorMsg(value="${o1.姓名} 开着 ${o2.车牌} 把 ${o3.车牌} 给撞了！",
        formatterClass=FreemarkerMessageFormatter.class)
    public static final int 撞车 = 5000000;


    //服务原子的实现中如下使用
    Assert.is(是否撞了, SomeErrorCodes.撞车, 司机, 车, 被撞的车);

```