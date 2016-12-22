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

## Boot And Handle Request
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
    protected function getHttpKernel()
    {
        return $this->container->get('http_kernel');
    }
    ...
}
```
应用内核的handle方法只有寥寥几行，它只做了两件事，调用自己的启动方法，然后从服务容器中取出 `http_kernel` 将Request交由其接管。从这里我们看出，其实Symfony的应用内核只是一个外壳，真正接管和处理请求的另有其人。这样做有一个很明显的优势：`http_kernel`服务可以被替换，可以扩展，可以自定义应用内核使用不同的服务，而所有的所有都是从容器开始，而应用内核在这里只是起到了一个初始化和引导启动的功能。那么我们就来看看默认的`http_kernel`是如何实现的，首先需要找到这个服务的配置文件：

```xml
// framework-bundle/Resources/config/services.xml

...
<service id="http_kernel" class="Symfony\Component\HttpKernel\HttpKernel">
    <argument type="service" id="event_dispatcher" />
    <argument type="service" id="controller_resolver" />
    <argument type="service" id="request_stack" />
    <argument type="service" id="argument_resolver" />
</service>
...

```
它来自Symfony的HttpKernel组件里的HttpKernel类，这个是官方实现的一个标准内核，当我们从服务容器中取出它时，是已经实例化的一个对象，在应用内核的handle方法中，初始化之后就调用了它的handle方法，让我们来一探handle方法：

```php
// Symfony/Component/HttpKernel/HttpKernel.php
class HttpKernel implements HttpKernelInterface, TerminableInterface
{
    ...
    public function handle(Request $request, $type = HttpKernelInterface::MASTER_REQUEST, $catch = true)
    {
        $request->headers->set('X-Php-Ob-Level', ob_get_level());
        try {
            return $this->handleRaw($request, $type);
        } catch (\Exception $e) {
            if ($e instanceof ConflictingHeadersException) {
                $e = new BadRequestHttpException('The request headers contain conflicting information regarding the origin of this request.', $e);
            }
            if (false === $catch) {
                $this->finishRequest($request, $type);
                throw $e;
            }
            return $this->handleException($e, $request, $type);
        }
    }
    ...
}
```

这个方法里是真正处理请求和框架内部抛出的异常的地方，直接调用handleRaw方法，如果它抛出异常，那么对异常进行处理并返回一个特殊的Response，这段代码你会发现它的解说其实就在文档。我们一点点来看吧：

```php
// Symfony/Component/HttpKernel/HttpKernel.php
class HttpKernel implements HttpKernelInterface, TerminableInterface
{
    ...
    private function handleRaw(Request $request, $type = self::MASTER_REQUEST)
    {
        $this->requestStack->push($request);
        // request
        $event = new GetResponseEvent($this, $request, $type);
        $this->dispatcher->dispatch(KernelEvents::REQUEST, $event);
        if ($event->hasResponse()) {
            return $this->filterResponse($event->getResponse(), $request, $type);
        }
        // load controller
        if (false === $controller = $this->resolver->getController($request)) {
            throw new NotFoundHttpException(sprintf('Unable to find the controller for path "%s". The route is wrongly configured.', $request->getPathInfo()));
        }
        $event = new FilterControllerEvent($this, $controller, $request, $type);
        $this->dispatcher->dispatch(KernelEvents::CONTROLLER, $event);
        $controller = $event->getController();
        // controller arguments
        $arguments = $this->argumentResolver->getArguments($request, $controller);
        $event = new FilterControllerArgumentsEvent($this, $controller, $arguments, $request, $type);
        $this->dispatcher->dispatch(KernelEvents::CONTROLLER_ARGUMENTS, $event);
        $controller = $event->getController();
        $arguments = $event->getArguments();
        // call controller
        $response = call_user_func_array($controller, $arguments);
        // view
        if (!$response instanceof Response) {
            $event = new GetResponseForControllerResultEvent($this, $request, $type, $response);
            $this->dispatcher->dispatch(KernelEvents::VIEW, $event);
            if ($event->hasResponse()) {
                $response = $event->getResponse();
            }
            if (!$response instanceof Response) {
                $msg = sprintf('The controller must return a response (%s given).', $this->varToString($response));
                // the user may have forgotten to return something
                if (null === $response) {
                    $msg .= ' Did you forget to add a return statement somewhere in your controller?';
                }
                throw new \LogicException($msg);
            }
        }
        return $this->filterResponse($response, $request, $type);
    }
    ...
    private function filterResponse(Response $response, Request $request, $type)
    {
        $event = new FilterResponseEvent($this, $request, $type, $response);
        $this->dispatcher->dispatch(KernelEvents::RESPONSE, $event);
        $this->finishRequest($request, $type);
        return $event->getResponse();
    }
    ...
    private function finishRequest(Request $request, $type)
    {
        $this->dispatcher->dispatch(KernelEvents::FINISH_REQUEST, new FinishRequestEvent($this, $request, $type));
        $this->requestStack->pop();
    }
    ...
}
```

从代码中不难看出，这里的实现和文档中的图例讲解一模一样：

![](/assets/01-workflow.png)

图中的Workflow便是此段代码的图示，其中的每个方块是一个事件

1. KernelEvents::REQUEST
2. KernelEvents::CONTROLLER
3. KernelEvents::CONTROLLER_ARGUMENTS
4. KernelEvents::VIEW
5. KernelEvents::RESPONSE
6. KernelEvents::FINISH_REQUEST

最后这些流程中一旦出现异常，会被外层捕获，并触发 `KernelEvents::FINISH_REQUEST`和`KernelEvents::EXCEPTION`，最后在`Front Controller`里调用应用内核的terminate方法触发`KernelEvents::TERMINATE`。

## Terminate
```php
// web/app.php
use Symfony\Component\HttpFoundation\Request;
...
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```
最终，当我们从应用内核获得一个响应之后，直接发送它，然后结束整个应用。

## 总结
这些事件点中除了必要的逻辑分支之外，只用了两行代码，如此简短的代码便是Symfony的核心流程，你可能这时会想到，其实Symfony的文档[从一开始](http://symfony.com/doc/current/introduction/http_fundamentals.html)就在灌输这种Request-Response生命周期的概念，因为它自始至终确实就是这么架构的。

Symfony用事件机制代替了硬编码的回调调用，而基于事件化整个框架，为Web开发添加了更灵活的架构方式，让程序更容易扩展，而且它提供了一个很棒的思路：**业务逻辑为何不以同样的思想来组织呢？**