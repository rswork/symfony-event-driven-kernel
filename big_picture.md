# Big Picture

通过分析Symfony的源代码，梳理顶层的运行逻辑，了解Symfony的最顶层事件。

## The Front Controller

```php
// web/app.php
use Symfony\Component\HttpFoundation\Request;
...

$kernel = new AppKernel('prod', false);
...

$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

[这段代码](https://github.com/symfony/symfony-standard/blob/3.2/web/app.php)精简的构建了Symfony框架的生命周期，所有框架内的功能都是由这段代码调用和执行的。

```
1. 实例化一个框架内核
2. 从php全局环境中创建Request
3. 将Request交给框架内核接管
4. 发送内核返回的Response内容
5. 内核结束一次“请求-响应”
```

## Boot

Symfony是从一个内核实例启动的，通过创建一个继承了 \`Symfony\Component\HttpKernel\Kernel\` 的实例对象启动一个Symfony应用。

## Handle Request

## Terminate

## 总结



