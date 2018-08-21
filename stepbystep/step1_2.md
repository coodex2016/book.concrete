# 1.2 实现api

建一个 `demo-impl` 子模块，模块目录为`demo/impl`

在pom中，引入以下依赖

{% github_embed "https://github.com/coodex2016/concrete-demo/blob/step1/demo/impl/pom.xml#L15-L34" %}{% endgithub_embed %}

> 同工程的模块间，推荐使用项目变量的方式指定`groupId`和`version`

- 依赖`javax.inject`（proviced），这是因为，基于java的，我们应该遵循java ee规范，通过规范剥离对技术组件的依赖。concrete的出发点也是这样 
- 依赖`demo-api`，因为这个模块要实现它
- 依赖`concrete-core`，因为`concrete-core`提供了很多好用的东西，在这一步虽然用不到，先放着吧

> 关于依赖的顺序，考虑到maven依赖的就近原则，为了尽可能减少不通作用域的依赖冲突，建议按照`test` `scope` `complie` `system`的作用域顺序维护依赖

建一个实现类 `org.coodex.concrete.demo.impl.DemoServiceImpl`

> 关于包名，建议所有的实现且与实现相关的内容都放在 `**.模块.impl` 下，比如 `**.模块.impl.copier` `**.模块.impl.listener` `**.模块.impl.interceptor`等

{% github_embed "https://github.com/coodex2016/concrete-demo/blob/step1/demo/impl/src/main/java/org/coodex/concrete/demo/impl/DemoServiceImpl.java" %}{% endgithub_embed %}

[1.3 使用Spring boot 跑起来](step1_3.md)