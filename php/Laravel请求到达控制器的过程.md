### 入口
Laravel5.8 入口文件为 public/index.php

```php
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );
    
    $response->send();
    
    $kernel->terminate($request, $response);
```

创建了一个 Kernel 对象，调用 handler 处理请求，获取返回结果。将返回结果输出到客户端，处理 terminate 操作。

### Kernel 中如何处理请求
容器里绑定的是 App\Http\Kernel, 继承于 Illuminate\Foundation\Http\Kernel。
```php
    /**
     * Handle an incoming HTTP request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();

            $response = $this->sendRequestThroughRouter($request);
        } catch (Exception $e) {
            $this->reportException($e);

            $response = $this->renderException($request, $e);
        } catch (Throwable $e) {
            $this->reportException($e = new FatalThrowableError($e));

            $response = $this->renderException($request, $e);
        }

        $this->app['events']->dispatch(
            new Events\RequestHandled($request, $response)
        );

        return $response;
    }
```

Kernel 中调用 sendRequestThroughRouter 方法，将请求传递到路由处理当中。

```php
    /**
     * Send the given request through the middleware / router.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);

        Facade::clearResolvedInstance('request');

        $this->bootstrap();

        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }

```

在 sendRequestThroughRouter 当中，在 app 中绑定了 request 实例，并解绑掉其他 request 实例对象。这样在程序其他地方都能通过 app()->make('request') 获取到 request 实例对象。

调用 bootstrap 方法，加载引导类。

创建一个 Pipeline 对象，将路由调度与中间件放入调用链当中。所有 request 先经过全局的中间件，然后在通过路由分发。

```php
    /**
     * Get the route dispatcher callback.
     *
     * @return \Closure
     */
    protected function dispatchToRouter()
    {
        return function ($request) {
            $this->app->instance('request', $request);

            return $this->router->dispatch($request);
        };
    }
```

因为 Piepline 调用链都是一个个的回调方法，所以在 dispatchToRouter 返回了一个匿名回调函数。使用 Kernel 的 route 属性进行调度。

Kernel 的 route 是一个 Illuminate\Routing\Router 对象。

### 路由调度

//Illuminate\Routing\Router

```php
    /**
     * Dispatch the request to the application.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response|\Illuminate\Http\JsonResponse
     */
    public function dispatch(Request $request)
    {
        $this->currentRequest = $request;

        return $this->dispatchToRoute($request);
    }

    /**
     * Dispatch the request to a route and return the response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response|\Illuminate\Http\JsonResponse
     */
    public function dispatchToRoute(Request $request)
    {
        return $this->runRoute($request, $this->findRoute($request));
    }

    /**
     * Return the response for the given route.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Routing\Route  $route
     * @return \Illuminate\Http\Response|\Illuminate\Http\JsonResponse
     */
    protected function runRoute(Request $request, Route $route)
    {
        $request->setRouteResolver(function () use ($route) {
            return $route;
        });

        $this->events->dispatch(new Events\RouteMatched($route, $request));

        return $this->prepareResponse($request,
            $this->runRouteWithinStack($route, $request)
        );
    }
```
从上面的方法可以看出，最终通过 findRoute 查找当前匹配的路由对象，并调用 runRoute 处理请求返回结果。

#### 怎么找到路由的

//Illuminate\Routing\Router
```php
    /**
     * Find the route matching a given request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Routing\Route
     */
    protected function findRoute($request)
    {
        $this->current = $route = $this->routes->match($request);

        $this->container->instance(Route::class, $route);

        return $route;
    }
```
对路由的匹配，是通过 routes 这个路由 Collections 去匹配的。

```php
    //Illuminate\Routing\RouteCollection

    /**
     * Find the first route matching a given request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Routing\Route
     *
     * @throws \Symfony\Component\HttpKernel\Exception\NotFoundHttpException
     */
    public function match(Request $request)
    {
        $routes = $this->get($request->getMethod());

        // First, we will see if we can find a matching route for this current request
        // method. If we can, great, we can just return it so that it can be called
        // by the consumer. Otherwise we will check for routes with another verb.
        $route = $this->matchAgainstRoutes($routes, $request);

        if (! is_null($route)) {
            return $route->bind($request);
        }

        // If no route was found we will now check if a matching route is specified by
        // another HTTP verb. If it is we will need to throw a MethodNotAllowed and
        // inform the user agent of which HTTP verb it should use for this route.
        $others = $this->checkForAlternateVerbs($request);

        if (count($others) > 0) {
            return $this->getRouteForMethods($request, $others);
        }

        throw new NotFoundHttpException;
    }

    /**
     * Determine if a route in the array matches the request.
     *
     * @param  array  $routes
     * @param  \Illuminate\Http\Request  $request
     * @param  bool  $includingMethod
     * @return \Illuminate\Routing\Route|null
     */
    protected function matchAgainstRoutes(array $routes, $request, $includingMethod = true)
    {
        [$fallbacks, $routes] = collect($routes)->partition(function ($route) {
            return $route->isFallback;
        });

        return $routes->merge($fallbacks)->first(function ($value) use ($request, $includingMethod) {
            return $value->matches($request, $includingMethod);
        });
    }
```

先通过请求的方法获取当前方法下可用的路由集合，在从这些集合中去遍历获取第一个匹配的路由。集合中每个 item 是一个 Illuminate\Routing\Router 对象。因此最终判断路由与请求是否匹配调用的是 Illuminate\Routing\Router 中的 matches 方法。

```php

    //Illuminate\Routing\Router

    /**
     * Determine if the route matches given request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  bool  $includingMethod
     * @return bool
     */
    public function matches(Request $request, $includingMethod = true)
    {
        $this->compileRoute();

        foreach ($this->getValidators() as $validator) {
            if (! $includingMethod && $validator instanceof MethodValidator) {
                continue;
            }

            if (! $validator->matches($this, $request)) {
                return false;
            }
        }

        return true;
    }

    /**
         * Get the route validators for the instance.
         *
         * @return array
         */
        public static function getValidators()
        {
            if (isset(static::$validators)) {
                return static::$validators;
            }
    
            // To match the route, we will use a chain of responsibility pattern with the
            // validator implementations. We will spin through each one making sure it
            // passes and then we will know if the route as a whole matches request.
            return static::$validators = [
                new UriValidator, new MethodValidator,
                new SchemeValidator, new HostValidator,
            ];
        }

```
在 Illuminate\Routing\Router 提供了四个默认的验证器，当四个验证器通过的时候才会匹配成功。四个验证器分别是 UriValidator 验证访问路径，MethodValidator 验证请求方法，SchemeValidator 验证访问协议，HostValidator 验证域名。其中对 uri 的验证内部是使用正则表达式验证。

### 路由调度怎么处理请求

```php
    //Illuminate\Routing\Router

    /**
     * Run the given route within a Stack "onion" instance.
     *
     * @param  \Illuminate\Routing\Route  $route
     * @param  \Illuminate\Http\Request  $request
     * @return mixed
     */
    protected function runRouteWithinStack(Route $route, Request $request)
    {
        $shouldSkipMiddleware = $this->container->bound('middleware.disable') &&
                                $this->container->make('middleware.disable') === true;

        $middleware = $shouldSkipMiddleware ? [] : $this->gatherRouteMiddleware($route);

        return (new Pipeline($this->container))
                        ->send($request)
                        ->through($middleware)
                        ->then(function ($request) use ($route) {
                            return $this->prepareResponse(
                                $request, $route->run()
                            );
                        });
    }

    /**
     * Run the route action and return the response.
     *
     * @return mixed
     */
    public function run()
    {
        $this->container = $this->container ?: new Container;

        try {
            if ($this->isControllerAction()) {
                return $this->runController();
            }

            return $this->runCallable();
        } catch (HttpResponseException $e) {
            return $e->getResponse();
        }
    }
```
路由对请求的处理也是返回一个 Pipeline, 先将请求通过中间件，然后在执行路由的 run 方法。在 run 方法里面判断当前是执行控制器方法还是回调方法，根据不同类型分开执行。

### 怎么执行
```php
    /**
     * Checks whether the route's action is a controller.
     *
     * @return bool
     */
    protected function isControllerAction()
    {
        return is_string($this->action['uses']);
    }

    /**
     * Run the route action and return the response.
     *
     * @return mixed
     */
    protected function runCallable()
    {
        $callable = $this->action['uses'];

        return $callable(...array_values($this->resolveMethodDependencies(
            $this->parametersWithoutNulls(), new ReflectionFunction($this->action['uses'])
        )));
    }

    /**
     * Run the route action and return the response.
     *
     * @return mixed
     *
     * @throws \Symfony\Component\HttpKernel\Exception\NotFoundHttpException
     */
    protected function runController()
    {
        return $this->controllerDispatcher()->dispatch(
            $this, $this->getController(), $this->getControllerMethod()
        );
    }
```
通过当前路由的 action 配置判断是否是控制器或者回调方法。从代码中可以看到，其实就是我们路由配置中的第二个参数对应到 action['user']。当我们第二参数是一个字符串的时候则认为是控制器方法，将请求转发到控制器里去处理。否则执行回调函数处理。

到这里，我们的请求就真的到达了我们的控制器的方法中，开始执行我们写的代码了。