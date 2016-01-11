# 核心组件
了解核心的构成组件实现的功能，学习如何单独使用它们，为了解Symfony的事件核心做准备。

Symfony的核心由几个组件组成：HttpFoundation、HttpKernel、DependencyInjection、EventDispatcher等。这些组件实现了Symfony的事件驱动核心的大部分功能。

## HttpFoundation
Symfony的[`HttpFoundation`](http://symfony.com/doc/current/components/http_foundation/index.html)组件对PHP默认的一些全局参数以及生成响应数据的一些函数做了一层面向对象方式的封装，替代原来的PHP对`Request`和`Response`的操作方式。本段大部分摘抄自[官方文档](http://symfony.com/doc/current/components/http_foundation/introduction.html)。

组件针对`Request`中的`HTTP请求头`、`URL数据`、`POST数据`、`COOKIE数据`、`SESSION据`、`文件上传`和`SERVER参数`进行了封装，可以通过`Symfony\Component\HttpFoundation\Request`类的实例化对象取得这些参数。

组件针对`Response`进行了统一的封装处理，使原本PHP散乱的无法管理各处响应的问题得以解决，通过构建一个`Symfony\Component\HttpFoundation\Response`对象来统一构建和返回响应数据。可以控制`响应的HTTP头`、`HTTP状态码`、`响应内容`、`COOKIE`等等