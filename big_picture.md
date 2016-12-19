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

[这段代码](https://github.com/symfony/symfony-standard/blob/v3.2.1/web/app.php)精简的构建了Symfony框架的生命周期，所有框架内的功能都是由这段代码调用和执行的。

```
1. 实例化一个框架内核
2. 从php全局环境中创建Request
3. 将Request交给框架内核接管
4. 发送内核返回的Response内容
5. 内核结束一次“请求-响应”
```

## Boot

Symfony是从一个内核实例接管一个请求来启动的，通过创建一个继承了 `Symfony\Component\HttpKernel\Kernel` 的实例对象初始化一个Symfony应用。一个Symfony应用内核包含了一些当前运行的环境参数和注册的bundle参数以及一些其它的和应用相关的属性和方法构成，它本身不是运行具体处理请求逻辑的对象，而是一个构建运行环境的对象：bundle系统的初始化，服务容器的初始化等等。

[默认的应用内核构造方法](https://github.com/symfony/symfony/blob/v3.2.1/src/Symfony/Component/HttpKernel/Kernel.php#L71)：

```php
// Symfony/Component/HttpKernel/Kernel.php

abstract class Kernel implements KernelInterface, TerminableInterface
{
    ...
    public function __construct($environment, $debug)
    {
        $this->environment = $environment;
        $this->debug = (bool) $debug;
        $this->rootDir = $this->getRootDir();
        $this->name = $this->getName();
        
        if ($this->debug) {
            $this->startTime = microtime(true);
        }
    }
    ...
}
```

当实例化一个应用内核时（new AppKernel\('prod', false\)）:

```php
    ...
    public function handle(Request $request, $type = HttpKernelInterface::MASTER_REQUEST, $catch = true)
    {
        if (false === $this->booted) {
            $this->boot();
        }
        
        return $this->getHttpKernel()->handle($request, $type, $catch);
    }
    ...
    public function boot()
    {
        if (true === $this->booted) {
            return;
        }
        
        if ($this->loadClassCache) {
            $this->doLoadClassCache($this->loadClassCache[0], $this->loadClassCache[1]);
        }
        
        // init bundles
        $this->initializeBundles();
        
        // init container
        $this->initializeContainer();
        
        foreach ($this->getBundles() as $bundle) {
            $bundle->setContainer($this->container);
            $bundle->boot();
        }
        
        $this->booted = true;
    }
    ...
}
```
应用内核的handle方法只有寥寥几行，它只做了两件事，调用自己的启动方法，然后从服务容器中取出 `http_kernel` 将Request交由其接管。从这里我们看出，其实Symfony的应用内核只是一个外壳，真正接管和处理请求的另有其人

## Handle Request

## Terminate

## 总结



