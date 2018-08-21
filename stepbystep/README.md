# 从0开始，通过实践了解一下concrete

在开始之前，需要掌握一些maven的基础知识。

本过程所有的代码放在了github上，地址 https://github.com/coodex2016/concrete-demo 不同的过程使用tag进行了标记

## 创建一个maven工程

我们以工程名 `concrete-demo` 为例。

## 建一个 `demo-api` 子模块

模块目录为`demo/api`

> why ?  
> 0.2.3开始，concrete注重系统逻辑上的模块化，
> concrete的一个模块颗粒度与maven的颗粒度不同，
> 一般来讲，一个逻辑上的concrete模块至少会包含`api` `impl`两个maven模块，
> `api`用来定义规范服务，`impl` 对 `api` 进行实现，把这二者分离开的原因是，`api`的调用方不依赖`impl`

## 在concrete-demo的pom.xml中增加依赖管理

> 使用快照版时，请先在pom或者maven的config增加sonatype的快照库，或者在私服中增加到sonatype快照库的代理

{% github_embed "https://github.com/coodex2016/concrete-demo/blob/step1/pom.xml#L17-L31" %}{% endgithub_embed %}


考虑到api包的兼容性，建议增加以下build参数

{% github_embed "https://github.com/coodex2016/concrete-demo/blob/step1/pom.xml#L34-L46" %}{% endgithub_embed %}

注意一下，所有源代码必须使用`UTF-8`，不要问我为什么

[1.1 定义api](step1_1.md)