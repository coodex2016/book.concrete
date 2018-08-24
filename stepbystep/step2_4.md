# 2.4 生成B端代码

我们借助spring-boot-start-web和jQuery来演示一下

## 1. 生成jQuery的代码

在`api-helper`模块中增加生成jQuery调用服务的代码

XGen.java
```java
        //生成jQuery调用服务的代码
        API.generateFor("demo.jquery",
                DemoService.class.getPackage().getName());
```

一样的，建好自己的`api_gen.demo.jquery.properties`，渲染描述符为`JaxRS.code.jquery.js.v1`

```properties
desc=JaxRS.code.jquery.js.v1
path=$$$YOUR PATH, Change it$$$
```

这样会生成一个文件`jquery-concrete.js`，一会用。

## 2. 简单改造release，演示生成的脚本功能

`demo-release`模块增加`spring-boot-starter-web`

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

> #### Hint::
>
> 这个依赖让`SpringBootServletInitializer`具备发布静态页面的功能（我只用到的这个，说的不对的，别打），默认使用resources下的`static`目录为静态页面的根

让`DemoBoot`继承自`SpringBootServletInitializer`
```java
public class DemoBoot extends SpringBootServletInitializer{
    // 原有代码，不用改动
}
```

新建resources/static/jquery/index.html
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

            var exampleModule = concrete.module("ExampleApi");

            $('#add45').click(function () {
                exampleModule.add(4, 5).then(function (sum) {
                    alert("4 + 5 = " + sum);
                })
            });

            $('#sayHello').click(function () {
                exampleModule.sayHello('jQuery').then(function (value) {
                    alert(value);
                })
            })
        });
    </script>
</head>
<body>
<button id="add45">add(4, 5)</button>
<button id="sayHello">sayHello('jQuery')</button>
</body>
</html>
```

把生成的文件拷贝到`resources/static/jquery/`目录下，跑起来，http://localhost:8080/jquery/index.html

需要编写的代码完全按照接口的定义来调用即可，剥离了`url`/`method`等多种技术细节，对于B端开发者来讲，只需要往接口里传数据就好了，把更多的精力放到人机交互的实现上。

截止到目前为止，我们掌握了基本的定义服务、文档化服务、发布服务、调用服务能力，后续小节我们开始介绍一些concrete的进阶能力。

截止到当前的代码： https://github.com/coodex2016/concrete-demo/tree/step2_4

> #### Hint::
>
> cocnrete生成前端代码不止支持jQuery的jaxrs调用
>
> | 渲染描述符 | 作用 |
> | -------- | ---- |
> | JaxRS.code.angular.ts.v1 | 生成ts的接口定义代码包，基于Observable模式，Angular 2<=v<4.3使用 |
> | JaxRS.code.angular.ts.v2 | 生成ts的接口定义代码包，基于Observable模式，推荐Angular版本高于4.3使用 |
> | WebSocket.code.jquery.js.v1 | Websockt服务端，生成基于jquery的javascript api |
> | WebSocket.code.angular.ts.v1 | Websockt服务端，生成ts的接口定义代码包，基于Observable模式 |
>
> 这些生成的代码，面向程序开发者是接口一致的。当服务端切换发布方式时，业务代码不需要做任何调整。