# org.coodex.concrete.common.IF

`IF`在`concrete`中用来做断言，如果符合，则会报指定异常

使用模式为

```txt
IF.逻辑(表达式, code, 渲染数据)
```

- IF.is(boolean, int, Object...)

如果表达式为真，则报参数2异常，并用可变参数3作为渲染的数据

- IF.isNull(T, int, Object...)

如果参数1为null，则报参数2异常，并用可变参数3作为渲染的数据，否则返回参数1

- IF.not(boolean, int, Object...)

如果表达式为假，则报参数2异常，并用可变参数3作为渲染的数据

- IF.notNull(Object, int, Object...)

如果参数1不为null，则报参数2异常，并用可变参数3作为渲染的数据
