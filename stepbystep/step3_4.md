# 3.4 异常信息

之前的步骤和大家自己的实践过程中，肯定碰到了很多异常，而调用端拿到的异常信息很不友好，这一小节我们就针对这点进行完善。

我们先规划一下怎么演示：

DemoService.add，我们约定它只能做加数被加数都小于10大于等于0的加法，超出范围的报个错

成了，就这样，开始动手。

## 定义错误号

异常信息属于设计的部分，所以，我们把这部分内容放在api模块中

为了方便演示，**我们去掉`DemoService`.`add`接口上的`@AccessAllow`注解**，即不登录也可正常执行

**新建**`org.coodex.concrete.demo.api.DemoErrorCodes`
```java
package org.coodex.concrete.demo.api;

import org.coodex.concrete.common.AbstractErrorCodes;

public class DemoErrorCodes extends AbstractErrorCodes {
    protected final static int DEMO_BASE = CUSTOM_LOWER_BOUND;

    public final static int TOO_HARD = DEMO_BASE + 1;
}
```

本例中，错误号`TOO_HARD`实际上是100001

> #### Hint:: 
>
> 我们推荐在注册服务时按包注册，这样，同包下的ErrorCodes类也可以被注册

resources/errorMsg目录下建两个资源文件，`demo.properties`和`demo_zh_CN.properties`

`demo.properties`
```properties
message.100001={0} + {1} is too hard!
```

`demo_zh_CN.properties`
```properties
message.100001={0} + {1} 太难了!
```

> #### Hint::
>
> 两个文件是为了演示服务端i18n的支持，一会会看到。如果你的IDE不支持properties自动转码，那么`demo_zh_CN.properties`使用这一段
> ```properties
> message.100001={0} + {1} \u592A\u96BE\u4E86!
> ```

以上，我们为`TOO_HARD`这个错误号配了一个zh_CN请求的错误信息和一个默认语言环境的错误信息

## 修改实现

`DemoServiceImpl`修改add方法实现
```java
    @Override
    public int add(int x1, int x2) {
        // step 3.4
        IF.is(x1 > 10 || x2 > 10 || x1 < 0 || x2 < 0, TOO_HARD, x1, x2);
        return x1 + x2;
    }
```

> #### Hint:: IF
>
> `IF`是concrete提供的一个断言工具类，我们所用到的接口原型是`IF.is(booleanExp, code, params)`，
> 意为，当`booleanExp`为真时，说明出现了错误号为`code`的异常，
> 并把params作为该异常的关键信息传递过去。
>
> `IF`中还有一些其他的接口，方便使用，例如：
> ```java
> Girl girl = IF.isNull(getGirlFromDataBase(), code);
> ```
> 表示尝试从数据库里获取Girl实例，如果获取为空，则产生code号错误，否则，给girl赋值

## 修改调用者

我们通过之前的jQuery演示来调用。

先用`XGen`重新生成`concrete-jquery.js`，拷贝到`release`模块的`resources/static/jquery`下

修改`index.html`
```html
<!DOCTYPE>
<html>
<head>
    <title>jQuery SDK example</title>
    <script type="text/javascript" rel="script"
            src="https://cdnjs.cloudflare.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script src="jquery-concrete.js"></script>
    <script>
        $(document).ready(function () {
            // 设置服务根
            concrete.configure({
                root: '/jaxrs'
            });

            var exampleModule = concrete.module("DemoService");

            $('#add45').click(function () {
                exampleModule.add(4, 5).then(function (sum) {
                    alert("4 + 5 = " + sum);
                })
            });
            // step 3.4
            $('#add145').click(function () {
                exampleModule.add(14, 5).then(function (sum) {
                    alert("14 + 5 = " + sum);
                })
            });

            $('#sayHello').click(function () {
                exampleModule.sayHello('jQuery').then(function (value) {
                    alert(value);
                })
            });

        });
    </script>
</head>
<body>
<button id="add45">add(4, 5)</button>
<!-- step 3.4 -->
<button id="add145">add(14, 5)</button>
<button id="sayHello">sayHello('jQuery')</button>
</body>
</html>
```

增加有step3.4注释的两段代码。按照这一套修改下来，不难理解，当我们点击`add(14,5)`的按钮时，服务端会产生一个异常。

我们继续

## 发布配置

在发布时，我们需要知道当前服务都用到了那些错误信息资源

`concrete.properties`
```properties
messagePattern.resourceBundles=errorMsg/demo
```

> #### Hint::
>
> concrete.properties是一个关键的配置文件，服务发布、系统配置、客户端环境等有很多配置项都在这定义，后续章节也会遇到
>
> messagePattern.resourceBundles用来指定默认情况下，错误信息模板使用的bundles，使用`,`分隔多个bundle

好，把`DemoBoot`跑起来。

访问 http://localhost:8080/jquery/index.html

点一下`add(14,5)`按钮看看，这个提示友好点了吧。

> #### Hint::服务端 i18n
>
> 我不确定你的浏览器语言环境是什么，但是，我们可以调整浏览器语言环境，
> 看看把zh_CN放在第一和en放在第一时，点击`add(14,5)`按钮时是否会有不同的信息。
>
> concrete的服务发布能够根据客户端的语言环境对资源文件进行选择，达到服务端的i18n。

这不是全部，关于error message，concrete也有几个扩展点
- `org.coodex.concrete.common.MessageFormatter`，如何格式化消息，默认是基于javaTextFormat
- `org.coodex.concrete.common.MessagePatternLoader`，如何加载错误信息模板，默认基于ResourceBundle

我们还可以通过`@ErrorMsg`注解对错误号进行装饰

截止到目前，代码版本为： https://github.com/coodex2016/concrete-demo/tree/step3_4