# Symfony's Event Driven Kernel

总结Symfony的事件驱动核心，如何在很短的代码行数上实现更多的扩展性。

## 核心组件
了解核心的构成组件实现的功能，学习如何单独使用它们，为了解Symfony的事件核心做准备。

## Big Picture
通过代码执行时的调用分支，抽取顶层的运行逻辑，了解Symfony的最顶层事件。

## 应用事件系统
了解如何运用Symfony标准的事件系统，做到对Request-Response控制自如。了解如何实现自己的事件，用自定义事件解耦业务逻辑中的复杂流程。

## 与经典MVC框架的对比
与经典MVC框架的粗略对比，主要指初始化、加载具体逻辑处理代码、发送响应完成请求过程中的差异。

## DIY事件驱动框架
如何DIY一个事件驱动的小框架，以Silex为例，使用Symfony的标准库编写一个小的玩具框架。