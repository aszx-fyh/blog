# Laravel 幕后：生命周期-路由

**本文翻译自 [LARAVEL BEHIND THE SCENES](https://crnkovic.me/tag/laravel-behind-the-scenes)** 

>    In this part, we will take a look at how Laravel runs through the middleware, how it matches the route against the request and how it executes the controller for your route. We will wrap up the "Lifecycle" series with creating a response, setting up headers, etc. But now, let's start with part 3. 
>
>  *在本部分中，我们将了解Laravel如何通过中间件运行，如何根据请求匹配路由，以及如何执行路由的控制器。我们将通过创建响应、设置头文件等来结束“生命周期”系列。现在，让我们从第3部分开始。*



## Review

> In [part 1](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-container), we reviewed what happens as soon as `index.php` file loads up, even before core framework services load. We saw how Laravel registers it's necessary service providers, such as router, logger and event dispatcher into the Container. In [part 2](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-boot-the-framework), we reviewed how the framework loads the configuration, sets up error handling, registers all service providers and resolves the facades. To freshen up your memory, that happened at the `handle` method of the Kernel class by [calling](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Foundation/Http/Kernel.php#L146) `bootstrap` method. We stopped there, and the unexamined code was this: 
>
>*在第1部分中，我们回顾了index.php文件加载后会发生什么，甚至是在核心框架服务加载之前。我们看到Laravel如何将必要的服务提供者(如路由器、日志记录器和事件分配器)注册到容器中。在第2部分中，我们回顾了框架如何加载配置、设置错误处理、注册所有服务提供者和解析`facade`。为了更新内存，通过调用`bootstrap`方法在内核类的句柄方法上发生了这样的事情。我们停在那里，未经检查的代码是这样的：* 



```php
//  Illuminate\Foundation\Http\Kernel sendRequestThroughRouter
return (new Pipeline($this->app))
    ->send($request)
    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
    ->then($this->dispatchToRouter());
```

## The stack of middleware

> First of all, we can try to simplify this code by trying to figure out when does the `shouldSkipMiddleware()` evaluate to true. This method [evaluates to true](https://github.com/laravel/framework/blob/3c11e7acc6ad14dd86e2f9e4bf3230010c3e611c/src/Illuminate/Foundation/Application.php#L850) if string literal `middleware.disable` is bound to the Container and is true (think of it as a global configuration). Note that this string it's set when you import the `WithoutMiddleware` trait in your tests and only then. We won't go deeper as this isn't in the scope of this lesson, but if you want to know more, [here's](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Foundation/Testing/Concerns/MakesHttpRequests.php#L93) the code behind it. Now that we know that empty array is passed only in tests, we know what we're passing to the `through` method: a list of our global middleware: 
>
> *首先，我们可以尝试通过计算`shouldSkipMiddleware()`的值何时为true来简化这段代码。如果字符串文字中间件，则此方法计算为true。`middleware.disable`绑定到容器，并且为true(将其视为全局配置)。注意，这个字符串是在测试中导入`WithoutMiddleware`特性时设置的。我们不会深入讨论，因为这不在本课的范围内，但是如果您想了解更多，这里是它背后的代码。现在我们知道空数组只在测试中传递，我们知道我们传递给`through`方法的是什么:我们的全局中间件列表:*



```php
array:6 [▼
  0 => "Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode"
  1 => "Illuminate\Foundation\Http\Middleware\ValidatePostSize"
  2 => "App\Http\Middleware\TrimStrings"
  3 => "Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull"
  4 => "App\Http\Middleware\TrustProxies"
  // other global middleware, such as InjectDebugbar from laravel-debugbar library
]
```

> We can also inline the `dispatchToRouter` method as it only returns a closure with little to no logic. Let's take a look at our simplified code now:
>
> *我们还可以内联`dispatchToRouter `方法，因为它只返回一个几乎没有逻辑的闭包。现在让我们来看看我们的简化代码*

```php
return (new Pipeline($this->app))
    ->send($request)
    ->through($this->middleware)
    ->then(function ($request) {
        $this->app->instance('request', $request);

        return $this->router->dispatch($request);
    });
```

> Now we can break this code down into few parts: **creating a Pipeline, passing the object through the stack and finally resolving the closure after the request has passed through the stack.**
>
>  *现在我们可以把这段代码分成几个部分:创建一个管道，通过堆栈传递对象，最后在请求通过堆栈后解决闭包。*

### The Pipeline-管道

> Think of Pipeline as a stack of classes. We take an object (in this case the **request**), modify it however we like it, then pass it onto the next class, and execute the same method as in the element before. This sounds familiar, right? When you create a new middleware in your App namespace, you have that `handle` method, remember? Well, this method accepts the request that was passed from the previous element in the stack and passes it on to the next element in that stack (`$next($request)`).
>
> *可以将管道看作类的堆栈。我们获取一个对象(在本例中是**request**)，按我们喜欢的方式修改它，然后将它传递给下一个类，并执行与前面元素相同的方法。这听起来很熟悉，对吧?当你在你的App命名空间中创建一个新的中间件时，你有那个`handle`方法，记得吗?这个方法接受从堆栈中的前一个元素传递过来的请求，并将其传递给堆栈中的下一个元素(` $next($request)`)。* 

#### Use case

> Perfect use-case for the pipeline is running a request through the middleware. We take a request, pass it onto the first middleware class, modify it, then do the same thing on the next middleware class. If anything along the way fails, we just push the client back to the top of the stack, show the error and don't let him through.
>
>  *管道的完美用例是通过中间件运行请求。我们接受一个请求，将其传递给第一个中间件类，修改它，然后对下一个中间件类执行相同的操作。如果中途有任何失败，我们只需将客户端推回到堆栈的顶部，显示错误并不让他通过。*

#### The implementation

> [If we open up](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Pipeline.php) the Pipeline class, we see that this class, that's in the `Illuminate\Routing` namespace, [inherits the Pipeline](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Pipeline/Pipeline.php) from the `Illuminate\Pipeline` namespace. That is done because Routing Pipeline needs some modifications. After creating an instance of the class and passing the Container to the constructor, nothing yet has happened. Then, we assign the object that's going to be passed through the pipes using the `->send($passable)`, in this case that is our **request**. Then, we have to assign the stack our object will be sent through, in our case we pass on the global middleware list using `->through($pipes)`. **Note that nothing has been sent through yet, we're just assigning class properties.** This is where things get interesting -- `then()` method happens. Let's take a look at the [source](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-routing/(https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Pipeline/Pipeline.php#L96)) for that method:
>
>  *如果我们打开管道类，就会看到这个位于`Illuminate\Routing`名称空间中的类从`Illuminate\Pipeline`名称空间继承`Pipeline` 。这样做是因为Routing Pipeline需要一些修改。在创建类的实例并将容器传递给构造函数之后，还没有发生任何事情。然后，我们使用`->send($passable)`分配将要通过管道传递的对象，在本例中这是我们的请求。然后，我们必须分配我们的对象将通过的堆栈，在我们的例子中，我们使用`->through($pipes)`传递全局中间件列表。注意，还没有发送任何内容，我们只是分配类属性。这就是事情变得有趣的地方——`then()`方法发生了。让我们看一下该方法的源代码* 

```php
public function then(Closure $destination)
{
    $pipeline = array_reduce(
        array_reverse($this->pipes), $this->carry(), $this->prepareDestination($destination)
    );

    return $pipeline($this->passable);
}
```

> **Note that `$this->pipes` is the list of middleware, `$this->passable` is the request and `$destination` is the final closure, in our case we accept what's left of the request and pass it onto the Router.**
>
>  *注意,`$this->pipes`是中间件的列表,`$this->passable`是请求和`$destination`是最终的闭包,在我们的例子里,我们接受请求的左边并将其传递到路由器上*

> What we're doing in this function is running our request through each middleware, then evaluating final destination (dispatching to the router). We're utilising [native PHP function array_reduce](http://php.net/manual/en/function.array-reduce.php) for iterating through array of middleware with a closure. Since this is pretty complex logic, it would take a long time for me to break down each component and describe what it does, and we still have routing to examine, so let's move onto the Routing.
>
>  *我们在这个函数中所做的是通过每个中间件运行我们的请求,然后评估最终目的地(发送到路由器)。我们正在使用一个关闭的中间件数组来使用[本机PHP函数array_reduce](http://php.net/manual/en/function.arrayreduce . PHP)。因为这是相当复杂的逻辑,所以我需要很长时间才能分解每一个组件并描述它所做的事情,我们仍然有路由检查,所以让我们走到路由上。*

## Routing

> Now that we have sliced through the middleware, we have the modified request passing onto the router. [If you take a look](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php#L587) at the `->router->dispatch($request)` method (*this is called in the closure of the `then` method*), we see that we're delegating the `dispatchToRoute` method in the Router itself, and that method executes the `runRoute` method:
>
>  现在我们已经分割了中间件，我们已经将修改后的请求传递到路由器。[如果你看一看](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php它)”`->router->dispatch($request)`方法(这在`then` 方法的闭包中被调用),我们看到,我们委托路由器本身的`dispatchToRoute`方法,这方法执行`runRoute`方法: 

```php
public function dispatchToRoute(Request $request)
{
    return $this->runRoute($request, $this->findRoute($request));
}
```

> Before we execute the route action, we have to find route that matches our request URI. There are `requestUri` and `method` properties on our Request object that can help us do that. [In `findRoute` method](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php#L611), we're calling the `match` method from `RouteCollection` object which finds the route, maps it to the Route object and after we have done that, we're binding that Route object into the Container. **Note that `RouteCollection` is the class we instantiated in the constructor when we were registering core RoutingServiceProvider (look: [part 1](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-container)).**
>
>  *在执行路由操作之前，我们必须找到与请求URI匹配的路由。在我们的请求对象上有`requestUri `和`method `属性可以帮助我们实现这一点。在`findRoute`方法,我们调用`match`方法从`RouteCollection`对象,找到路线,地图的路线对象之后,我们所做的,我们这条路对象绑定到容器中。**请注意，`RouteCollection`是我们在注册core RoutingServiceProvider时在构造函数中实例化的类(参见:[part 1](https://crnkovic.me/laravel- es-lifecycle-container) )**。*

### Matching the route

The contents of the `match` method in `Illuminate\Routing\RouteCollection` [are as follows](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/RouteCollection.php#L157):

```php
public function match(Request $request)
{
    $routes = $this->get($request->getMethod());

    $route = $this->matchAgainstRoutes($routes, $request);

    if (! is_null($route)) {
        return $route->bind($request);
    }

    $others = $this->checkForAlternateVerbs($request);

    if (count($others) > 0) {
        return $this->getRouteForMethods($request, $others);
    }

    throw new NotFoundHttpException;
}
```

> `$this->get($request->getMethod())` grabs all the defined routes for the request method, e.g. if the request is of GET type, we return an array of all routes that can be accessed with the GET method. The structure of this array is defined as `uri => Route-object`.
>
>  *`$this->get($request->getMethod())`获取请求方法定义的所有路由，例如，如果请求是get类型的，我们返回一个数组，其中包含可以用get方法访问的所有路由。这个数组的结构被定义为`uri => Route-object`。* 

```php
array:27 [▼
  "api/user" => Route {#298 ▶}
  "/" => Route {#300 ▶}
  "login" => Route {#307 ▶}
  "register" => Route {#310 ▶}
  "password/reset" => Route {#312 ▶}
  "password/reset/{token}" => Route {#314 ▶}
]
```

> Then, we try to match request against `$routes` using the `matchAgainstRoute` method, and that method essentially loops through the array of routes and calls the `matches` method on Route object to see if the requests matches this route. The first route hit is returned back and stops the iteration ([utilizing `Collection@first` method](https://laravel.com/docs/5.6/collections#method-first)). **Note that we're pushing the [fallback routes](https://twitter.com/themsaid/status/910135205989625856) to the end of the collection using `partition` then `merge`. This is done because we first want to check regular routes, then fallback routes in case no regular has been found.**
>
>  *然后，我们尝试使用`matchAgainstRoute`方法将请求与`$routes`进行匹配，该方法实际上循环遍历路由数组并调用Route对象上的`matches `方法来查看请求是否与该路由匹配。第一个路由命中被返回并停止迭代([使用' Collection@first '方法](https://laravel.com/docs/5.6/collections#method-first))。注意，我们使用“分区”然后“合并”将[回退路由](https://twitter.com/themsay/status/910135205989625856)推到集合的末尾。这样做是因为我们首先要检查常规路由，然后在没有发现常规路由的情况下检查备份路由*

```php
// mark，重点查看
protected function matchAgainstRoutes(array $routes, $request, $includingMethod = true)
{
    list($fallbacks, $routes) = collect($routes)->partition(function ($route) {
        return $route->isFallback;
    });

    return $routes->merge($fallbacks)->first(function ($value) use ($request, $includingMethod) {
        return $value->matches($request, $includingMethod);
    });
}
```

> The `matches` method of the `Route` class resolves and executes different route validators: `URIValidator`, `SchemeValidator`, `HostValidator` & `MethodValidator`. Those validator classes try to validate the route for its specific purpose. If you want to know more, [take a look at the source code](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Route.php#L259). *Since this is pretty complex, maybe I'll dedicate a complete series of Laravel Behind the Scenes to Routing*. **Essentially, we're just mapping the URI of the request to any of the compiled routes, but note that we also need to match the request method.** If we have found any routes that match, we bind the request to the route and return it back. However, if no route was matched in the process, we will loop through the route collection to see if that URI exists for different HTTP methods using `checkForAlternateVerbs($request)`. If none exist, throw back an Exception.
>
>  *`Route`类的`matches`方法解析并执行不同的路由验证器:`URIValidator`、`SchemeValidator`、`HostValidator`和`MethodValidator`。这些验证器类尝试验证路由的特定用途。如果您想了解更多，[看一下源代码](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Route.php#L259)。由于这是相当复杂的，也许我将奉献一个完整的系列Laravel幕后路由。本质上，我们只是将请求的URI映射到任何已编译的路由，但是请注意，我们还需要匹配请求方法。如果找到匹配的路由，我们将请求绑定到路由并返回。但是，如果在过程中没有匹配路由，我们将循环遍历路由集合，查看使用`checkForAlternateVerbs($request)`的不同HTTP方法是否存在该URI。如果不存在，则抛出异常。*

### Executing the route

Now that we have found our route, we are ready to execute it. This is what `runRoute` method does. [If we take a look](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php#L627) at the definition of that method, we see a bunch:

```php
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

> **Note that we initially need to set the route resolver in the Request class, so when you use the `$request->route()` method in your classes, that resolver finds that route and returns it back to you.** Afterwards we're just firing off the event that we have matched the route, nothing special. Then, we "run route within stack". What does that mean? We need to take a look at the code to make more sense.
>
>  *注意，我们最初需要在请求类中设置路由解析器，因此当您在类中使用`$Request ->route() `方法时，该解析器会找到路由并将其返回给您。之后，我们只是发射我们匹配路线的事件，没有什么特别的。然后，我们“在堆栈中运行路由”。这是什么意思?我们需要看一下代码以使其更有意义。*

#### Route middleware

```php
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
```

> Yep, that code looks familiar -- the middleware. We again create a pipeline, but this time on we run the request through [middleware defined for this route](https://laravel.com/docs/5.6/middleware#assigning-middleware-to-routes). Since we already know what this code does, we should only take a look at the closure in `then` method, in this case:

```php
function ($request) use ($route) {
    return $this->prepareResponse(
        $request, $route->run()
    );
}
```

#### Dispatching the Controller

This is where we execute the route, dispatch the controller and return back the response to `prepareResponse`.

```php
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

> [Looking at this method](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Route.php#L163), we see that we just try to dispatch the controller (if the route was defined as a controller action), or execute the callback (if route was defined as a closure), or throw an error. `isControllerAction` checks if `uses` parameter in your routes is a string (e.g. `Route::get('/', 'SomeController@someMethod')`). If it is, execute the `runController` method. If it's not, execute the `runCallable` method which executes the closure-defined route. [Looking](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Route.php#L209) at the `runController`, we see that it does exactly that:
>
>  *[看这个方法](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Route.php # L163),我们看到,我们只是试图调度控制器(如果路线被定义为一个控制器动作),或执行回调(如果路线被定义为一个闭包),或者抛出一个错误。`isControllerAction `检查您的路由中的`uses`参数是否为字符串(例如`Route::get('/', 'SomeController@someMethod')`)。如果是，则执行`runController`方法。如果不是，则执行`runCallable`方法，该方法执行封闭定义的路由。 看看`runController`，我们看到它确实是这样做的:*

```php
protected function runController()
{
    return $this->controllerDispatcher()->dispatch(
        $this, $this->getController(), $this->getControllerMethod()
    );
}
```

> A lot of code happens in the background of this, so we won't go that deep. **Note: this is where all your controller dependencies are automatically injected.**
>
> So in essence, `$route->run()` executed your controller method defined for this route and got back some response (**note that this is the response you return in your controller action**) and that response output is passed on the `prepareResponse` method.
>
> *很多代码都是在后台发生的，我们就不深入了。注意:这里是所有控制器依赖项自动注入的地方*
>
> *因此，本质上，`$route->run()`执行为此路由定义的控制器方法，并返回一些响应(**注意，这是您在控制器操作**中返回的响应)，该响应输出传递给`prepareResponse`方法。*



### Response

The `prepareResponse` [delegates](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php#L697) the static `toResponse` method whose purpose is to create a new `Response` class and prepare the response for sending it back to the client. [That looks like this](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php#L709):

```php
public static function toResponse($request, $response)
{
    if ($response instanceof Responsable) {
        $response = $response->toResponse($request);
    }

    if ($response instanceof PsrResponseInterface) {
        $response = (new HttpFoundationFactory)->createResponse($response);
    } elseif ($response instanceof Model && $response->wasRecentlyCreated) {
        $response = new JsonResponse($response, 201);
    } elseif (! $response instanceof SymfonyResponse &&
               ($response instanceof Arrayable ||
                $response instanceof Jsonable ||
                $response instanceof ArrayObject ||
                $response instanceof JsonSerializable ||
                is_array($response))) {
        $response = new JsonResponse($response);
    } elseif (! $response instanceof SymfonyResponse) {
        $response = new Response($response);
    }

    if ($response->getStatusCode() === Response::HTTP_NOT_MODIFIED) {
        $response->setNotModified();
    }

    return $response->prepare($request);
}
```

> **Note that `$response` variable in the argument is the exact output we returned back from our controller.** This method is ready to modify that response however is necessary. We returned back the Model instance? No problem, check if `$response` is a model?, new-up the `JsonResponse` class and it's done. We returned the string? No problem, new up regular `Response` class and voila. We returned anything that's JSON serializable? Convert it to the `JsonResponse` object. **Also note that if you returned back Eloquent model AND that resource was just created (in this very request), Laravel will set status code of the response to 201.**
>
> *请注意，参数中的`$response`变量是我们从控制器返回的确切输出。此方法可以修改该响应，但是有必要。我们返回了Model实例？没问题，检查`$response`是否为模型？，对`JsonResponse`类进行新的修改并完成。我们返回了字符串？没问题，
> 更新常规的`Response`类，瞧。我们返回了JSON可序列化的任何内容？将其转换为`JsonResponse`对象。 还请注意，如果您返回了Eloquent模型，并且刚刚创建了该资源（在此请求中），Laravel会将响应的状态代码设置为201。*

------

> The only thing left in our lifecycle is to examine this `prepare($request)` method of our Response object and finish up the remaining code in `index.php` -- which sends the response to the client and executes any terminable middleware. In the next part, we will take a look at the Response class and dive deeper into the Symfony's Response component and probably wrap up the "Lifecycle" series.
>
>  在我们的生命周期中剩下的唯一一件事就是检查这个响应对象的`prepare($request)`方法，并完成`index.php`中的剩余代码。将响应发送到客户机并执行任何可终止的中间件。在下一部分中，我们将研究Response类并深入研究Symfony的响应组件，并可能结束“生命周期”系列。 