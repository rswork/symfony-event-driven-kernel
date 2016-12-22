# 核心组件
了解核心的构成组件实现的功能，学习如何单独使用它们，为了解Symfony的事件核心做准备。这一章只是简短介绍各个组件的功能，你需要自己引入这些组件进行相应的学习和练。

Symfony的核心由几个组件组成：HttpFoundation、HttpKernel、EventDispatcher、DependencyInjection等。这些组件实现了Symfony的事件驱动核心的大部分功能。

你可以通过[组件的官方文档](http://symfony.com/doc/current/components/index.html)来了解和使用它们，也可以根据文档编写一些测试脚本来熟悉组件的用法和用途，帮助自己更好的了解组件的功能。

## HttpFoundation
Symfony的[`HttpFoundation`](http://symfony.com/doc/current/components/http_foundation/index.html)组件对PHP默认的一些全局参数以及生成响应数据的一些函数做了一层面向对象方式的封装，替代默认的PHP对`Request`和`Response`的操作方式。本段大部分内容摘抄自[官方文档](http://symfony.com/doc/current/components/http_foundation/introduction.html)。

针对`Request`中的`HTTP请求头`、`URL数据`、`POST数据`、`COOKIE数据`、`SESSION据`、`文件上传`和`SERVER参数`进行了封装，可以通过`Symfony\Component\HttpFoundation\Request`类的实例化对象取得这些参数。对于[`Session`](http://symfony.com/doc/current/components/http_foundation/sessions.html)管理也同样进行了封装，统一处理。

针对`Response`进行了统一的封装处理，使原本PHP散乱的无法管理各处响应的问题得以解决，通过构建一个`Symfony\Component\HttpFoundation\Response`对象来统一管理响应数据。可以控制`响应的HTTP头`、`响应内容`、`COOKIE`等等。可以响应`html内容`、`JSON数据`、`托管静态文件`、`数据流形式响应`等等常见的给客户端响应的数据形式。

## HttpKernel
[`HttpKernel`](http://symfony.com/doc/current/components/http_kernel/index.html)组件提供了一个php框架所必须的基础，它是Symfony框架的基石，而且你还可以独立使用它来构建自己的框架。这个组件基于一套HTTP生命周期流程化的概念构建而成，流程主要关注如何将`Request`转换成`Response`，借助`EventDispatcher`组件，实现这个基于事件驱动的流程的核心，即整个HTTP生命周期流程化抽象的实现。

它基于的核心概念是在一个HTTP请求的生命周期中从`Request`到`Response`的流程化，即[`The Workflow of a Request`](http://symfony.com/doc/current/components/http_kernel/introduction.html#the-workflow-of-a-request)。另外它具有很高的可定制性，不但可以实现全栈型的重型框架\(Symfony\)，又可以实现微型框架\(Silex\)，或者是一个专业的CMS\(Drupal、eZPlatform\)。

它定义了一个HTTP请求的几个[`标准事件`](http://symfony.com/doc/current/components/http_kernel/introduction.html#component-http-kernel-event-table)，使框架结构清晰明了，并且给开发者提供了完善的扩展度很高的流程控制框架，带来了比MVC更自由的开发体验。

## EventDispatcher
[`EventDispatcher`](http://symfony.com/doc/current/components/event_dispatcher/introduction.html)组件是一个让项目具有更好的扩展性但不用修改原逻辑代码的工具，降低每个组件之间的耦合度，减少硬编码调用，使组件之间的调用只需要监听相应的事件和监听事件。Symfony的核心就是定义了一系列事件，触发，其余的框架组件只需要监听这些事件，多一个组件，只是多了一个监听，Kernel的代码没修改一行。

你可以在其它框架引入此组件，定义自己的流程并使用事件来组织各个模块的调用，使框架更灵活扩展，而且使流程控制代码和具体的处理逻辑分离，随时可进行模块的添加和升级。

## DependencyInjection
[`DependencyInjection`](http://symfony.com/doc/current/components/dependency_injection/introduction.html)是Symfony框架的基础，标准化和集中对象在应用程序中构造的方式。脱离框架它也能独立使用，因为它本身没有太多强制性的依赖。它提供了一个服务容器，将定义的各种对象注入到这个容器中，每次需要一个对象不用再手动实例化它，只需要从容器中取出即可使用。Symfony中大部分功能都会注入到服务容器中，事件系统也是基于服务容器构建，它是Symfony可灵活扩展的基础，为企业系统开发提供了更可靠的基础架构。

