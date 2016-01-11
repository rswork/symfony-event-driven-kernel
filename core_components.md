# 核心组件
了解核心的构成组件实现的功能，学习如何单独使用它们，为了解Symfony的事件核心做准备。

Symfony的核心由几个组件组成：HttpFoundation、HttpKernel、DependencyInjection、EventDispatcher等。这些组件实现了Symfony的事件驱动核心的大部分功能。

## HttpFoundation
Symfony的[`HttpFoundation`](http://symfony.com/doc/current/components/http_foundation/index.html)组件对PHP默认的一些全局参数以及生成响应数据的一些函数做了一层面向对象方式的封装，替代原来的PHP对`Request`和`Response`的操作方式。本段大部分摘抄自[官方文档](http://symfony.com/doc/current/components/http_foundation/introduction.html)。

组件针对Request中的HTTP请求头、URL数据、POST数据、COOKIE数据、文件上传和SERVER参数进行了封装，可以通过`Symfony\Component\HttpFoundation\Request`类的实例化对象取得这些参数，