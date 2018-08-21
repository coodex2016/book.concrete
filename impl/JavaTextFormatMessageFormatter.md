# 基于java.text.MessageFormat的MessageFormatter

详见[java.text.MessageFormat](http://docs.oracle.com/javase/8/docs/api/java/text/MessageFormat.html).

例如：

```java
    @ErrorMsg(value="“多少钱？” “2w。” “{0}, {0}, {0} .....{0}什么玩笑”")
    public static final int 结巴 = 5000000;


    //服务原子的实现中如下使用
    Assert.is(是否结巴, SomeErrorCodes.结巴, "开");

```