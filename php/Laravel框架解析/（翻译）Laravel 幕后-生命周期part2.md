# Laravel 幕后：生命周期-启动框架

**本文翻译自 [LARAVEL BEHIND THE SCENES](https://crnkovic.me/tag/laravel-behind-the-scenes)** 

>  I'm Josip and I've been working with Laravel for quite some time and I've decided to help out others figure out how this amazing framework works. During that process, I'll also show you some cool tricks that are happening behind the scenes. In this series of blog posts, I will try to explain how Laravel works under the hood, from getting a request in index.php to rendering the response back to the client. Since this is pretty advanced and thorough, I will split the guide into couple parts. In this part 2, I will try to explain how request is created, how global variables are read, and how service providers and facades are registered. Let's begin with part 2. 
>
>  **This guide covers the source code of version 5.6 of the Laravel framework.** 
>
>  *我是Josip，我和Laravel一起工作了很长一段时间，我决定帮助其他人弄清楚这个惊人的框架是如何工作的。在这个过程中，我还将向您展示幕后发生的一些很酷的技巧。在本系列博客文章中，我将尝试解释Laravel的工作原理，从在index.php中获取请求到将响应呈现回客户端。由于这是相当先进和深入的，我将把指南分为几个部分。在第2部分中，我将尝试解释如何创建请求、如何读取全局变量以及如何注册服务提供者和facade。让我们从第2部分开始。*
>
>  *本指南涵盖了Laravel框架5.6版本的源代码。*



## Binding the kernel-绑定 http kernel

>[In part 1](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-container), we reviewed what happens as soon as the Application instance is created and which core service providers are loaded. Next up is Kernel. As you can in `bootstrap/app.php`, after we new-up the Application, we try to bind a singleton of the HTTP and Console kernels to the Container。
>
>*我们回顾了在创建应用程序实例时发生的事情以及加载哪些核心服务提供者。下一个是Kernel。你可以在`bootstrap/app`。在更新应用程序之后，我们尝试将HTTP和控制台Kernel的单例绑定到容器。* 



```php
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);
```

> **Note that we're actually binding OUR own Kernel implementation that can be found in `app/Http/`.** However, you may not have noticed that our kernel inherits the `Illuminate\Foundation\Http\Kernel` class. After we have stored our kernel classes and exception handler in the Container, nothing yet has been changed. We have just saved them for later use. This is being done in the `app.php` instead of somewhere hidden in the `vendor` directory so we can change which Kernel implementation is bound. You can (but probably never will) swap `App\Http\Kernel` with any Kernel implementation you want. No kernel has yet been instantiated, no Kernel object has been created. This is what happens next. **Also note that we're binding the interface as a *key* to fetch from the Container, but we're actually resolving `App\Http\Kernel`.**
>
> *注意，我们实际上是在绑定我们自己的Kernel实现，它可以在app/Http/中找到。但是，您可能没有注意到，我们的Kernel继承了`Illuminate\Foundation\Http\Kernel`。在容器中存储了Kernel类和异常处理程序之后，没有东西被改变。我们只是把它们保存起来以后用。这是在app.php中完成的，而不是隐藏在vendor目录中，因此我们可以更改绑定的Kernel实现。您可以(但可能永远不会)使用任何您想要的Kernel实现替换App\Http\Kernel。还没有实例化Kernel，也没有创建Kernel对象。这就是接下来发生的事情。还要注意，我们将接口绑定为从容器中获取的键，但实际上我们正在解析App\Http\Kernel。*

​	

## Resolving the kernel-解析kernel

>The very next line in `index.php` is as follows:

```php
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
```

> This is the perfect example of interface binding, we bound Kernel interface, but actually are resolving concrete class implementation. This is where things get interesting. After we call `make()`, we can see that a new instance of the Kernel has been created and stored in the `$kernel` variable. Let's take a look at the constructor of the Kernel class (the class that `App\Http\Kernel` inherits from).
>
> *这是接口绑定的完美例子，我们绑定了Kernel接口，但实际上是在解析具体的类实现。这就是事情变得有趣的地方。在调用' ` make()`之后，我们可以看到已经创建了一个新的Kernel实例，并将其存储在`$Kernel`变量中。让我们来看看Kernel类(被App\Http\Kernel继承的类)的构造函数。*

```php
// Illuminate\Foundation\Http\Kernel
public function __construct(Application $app, Router $router)
{
    $this->app = $app;
    $this->router = $router;

    $router->middlewarePriority = $this->middlewarePriority;

    foreach ($this->middlewareGroups as $key => $middleware) {
        $router->middlewareGroup($key, $middleware);
    }

    foreach ($this->routeMiddleware as $key => $middleware) {
        $router->aliasMiddleware($key, $middleware);
    }
}
```

>**Note that `$app` and `$router` parameters in the constructor are being resolved and injected automatically.** We will take a look at the `make()` command in the *Who wants to know more* segment.
>
>**Also note that the `$router` here is just a new instance of the Router class and does not contain any routes or any configuration, just an Event dispatcher, that as you can remember, we bound to the Container in part 1.** If we dump the contents of the `$router` class, we can prove that to ourselves:
>
>*注意，构造函数中的`$app`和`$router`参数被自动解析和注入。我们将在**Who wants to know more**片段中查看make()命令。*
>
>*还要注意，这里的`$router`只是Router类的一个新实例，它不包含任何路由或任何配置，只是一个事件分派器，正如您所记得的，我们在第1部分中将其绑定到容器。如果我们输出`$router`类的内容，我们可以向自己证明:*



```php
Router {#36 ▼
  #events: Dispatcher {#37 ▶}
  #container: Application {#7 ▶}
  #routes: RouteCollection {#39 ▼
    #routes: []
    #allRoutes: []
    #nameList: []
    #actionList: []
  }
  #current: null
  #currentRequest: null
  #middleware: []
  #middlewareGroups: []
  +middlewarePriority: []
  #binders: []
  #patterns: []
  #groupStack: []
}
```

>Now, onto the code. After we have injected the Container and the Router, we're getting the list of global middleware and assigning them to the Router. **Note:** if you're wondering how can `$router->property = $contents` change the `$router` class contents (doesn't work with regular variables), remember that [PHP passes classes by reference](http://php.net/manual/en/language.references.pass.php) (meaning that if you modify the `$router` variable, the real class contents are being modified).
>
>We're also just getting middleware groups and route middleware defined in the `app/Http/Kernel.php` and assigning them to the class properties in Router, nothing more. Basically, Kernel constructor reads middleware list from your own Kernel class and passes them onto the Router, so Router can operate on it. **Note that route collection is still empty.**
>
>*现在来看代码。在我们注入容器和路由器之后，我们将获得全局中间件的列表并将它们分配给路由器。注意:如果您想知道`$router->property = $contents`如何更改`$router`类内容(不与常规变量一起工作)，请记住PHP通过引用传递类(这意味着如果您修改了`$router`变量，实际的类内容将被修改)。*
>
>*我们还获得了中间件组和路由中间件定义在`app/Http/Kernel.php`，并将它们分配到Router中的类属性，仅此而已。基本上，Kernel构造函数从您自己的Kernel类中读取中间件列表，并将它们传递给路由器，这样路由器就可以对其进行操作。注意，路由集合仍然是空的。*



## Capture the request-捕获请求

>  This is where real magic happens. Let's examine the following code snippet: 

```php
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
```

> You may think that `handle()` method is being called first, but no. First thing we do now is capture the request. Let's take a look at that capturing. 
>
> *您可能认为首先调用了`handle() `方法，但事实并非如此。我们现在要做的第一件事是捕获请求。让我们来看看这个捕捉。*

```php
public static function capture()
{
    static::enableHttpMethodParameterOverride();

    return static::createFromBase(SymfonyRequest::createFromGlobals());
}
```

> As we can see, this method creates a new Request instance from global variables and returns that object back. If we examine that, you notice this long-ass-named method called `enableHttpMethodParameterOverride`. You have probably used `method_field('DELETE')` in your forms in view files before, right? In `enableHttpMethodParameterOverride()`, we just set the same-named property to `true` to allow the Laravel to read `_method` from the request data to see what type of the request we actually want. Next up, we're actually utilizing Symfony's Request class from the [HttpFoundation](https://github.com/symfony/http-foundation) component to create a new Symfony Request from global variables (`$_SERVER, $_POST, $_GET, $_COOKIE, ...`). [Taking a look](https://github.com/symfony/http-foundation/blob/master/Request.php#L279) at the `createFromGlobals()` from the Symfony's Request class, we see the following: 
>
> *正如我们所看到的，这个方法从全局变量创建一个新的请求实例并返回该对象。如果我们对此进行研究，您会注意到这个名为`enableHttpMethodParameterOverride`的**long-ass-named**的方法。你可能曾经在你的视图文件的窗体中使用过`method_field('DELETE')`，对吗?在`enableHttpMethodParameterOverride()`中，我们只是将同名属性设置为“true”，以允许Laravel从请求数据读取“_method”，以查看我们实际需要的请求类型。接下来，我们实际上使用了来自[HttpFoundation](https://github.com/symfony/http-foundation)组件的Symfony Request class，从全局变量（$_SERVER，$_POST, $_GET, $_COOKIE, ...）创建一个新的[Symfony Request](https://github.com/symfony/http-foundation/blob/master/Request.php#L279)。在Symfony的请求类的`createFromGlobals()`中，我们看到了以下内容:*

```php
public static function createFromGlobals()
{
    $request = self::createRequestFromFactory($_GET, $_POST, array(), $_COOKIE, $_FILES, $_SERVER);

    if (0 === strpos($request->headers->get('CONTENT_TYPE'), 'application/x-www-form-urlencoded')
        && in_array(strtoupper($request->server->get('REQUEST_METHOD', 'GET')), array('PUT', 'DELETE', 'PATCH'))
    ) {
        parse_str($request->getContent(), $data);
        $request->request = new ParameterBag($data);
    }

    return $request;
}
```

> Basically, this code calls [another static method](https://github.com/symfony/http-foundation/blob/master/Request.php#L1907) called `createRequestFromFactory`, which in essence just returns the new instance of self: 

```
rivate static function createRequestFromFactory($bunchOfServerVariables)
{
    // ... something we don't care about at the moment

    return new static($bunchOfServerVariables);
}
```

> This is all being done in the Symfony's Request component and I will probably sometime dig into that. Symfony's Request class reads all the global variables (such as cookies, headers, server vars, ...) on initialization ([in a constructor](https://github.com/symfony/http-foundation/blob/master/Request.php#L233) and stores them. Remaining code from the `createFromGlobals` checks if our request type is PUT, DELETE or PATCH and just reads the input parameters. As of this moment, headers have been read, server variables have been read, cookies have been read. Let's dive back into our Laravel's `capture` method. Next thing is `createFromBase`, which accepts the request Symfony just created, duplicates all of its contents and creates a new instance of Laravel's implementation of the Request class. 
>
>  这些都是在Symfony的请求组件中完成的，我可能会深入研究这个问题。Symfony的请求类在初始化时读取所有全局变量(如cookie、header、server vars等)([在构造函数中](https://github.com/symfony/http-foundation/blob/master/Request.php#L233)并存储它们。`createFromGlobals`中的其余代码检查我们的请求类型是`PUT`、`DELETE`还是`PATCH`，并只读取输入参数。此时，已经读取了报头、服务器变量和cookie。让我们回到`Laravel`的`capture` 方法。接下来是`createFromBase`，它接受Symfony刚刚创建的请求，复制它的所有内容并创建一个新的Laravel实现请求类的实例 

```php
public static function createFromBase(SymfonyRequest $request)
{
    // Right now we don't care about this
    if ($request instanceof static) {
        return $request;
    }

    $content = $request->content;

    $request = (new static)->duplicate(
        $request->query->all(), $request->request->all(), $request->attributes->all(),
        $request->cookies->all(), $request->files->all(), $request->server->all()
    );

    $request->content = $content;

    $request->request = $request->getInputSource();

    return $request;
}
```

> As of this moment, we have a request information, such as request type, URI, headers, cookies, etc. You can examine that Request class by `dd($request)` when capturing.

## Kernel handling-Kernel处理过程

Next up, we see that we call `handle()` method on the request we just captured. Let's examine the `handle` method from our core Kernel class.

```php
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

> **Note that we are calling `enableHttpMethodParameterOverride()` once again, just to be sure that `_method` parameter can be read (in case Symfony changes theirs implementation)**. After that marked that flag, we pass the request through the router to create a response, we dispatch an event that we handled the request and return that response back to the user. If something along the way fails or throws an Exception, Laravel's Exception handler (that we bound in the `app.php`) will pick up and report/render that exception to the user. Taking a look at the `sendRequestThroughRouter` we see some cool things:
>
> ***注意，我们再次调用了`enableHttpMethodParameterOverride()`，只是为了确保`_method`参数可以被读取(以防Symfony更改了它们的实现)。****在标记了该标志之后，我们通过路由器传递请求来创建响应，我们分派一个处理请求的事件并将响应返回给用户。如果中途发生故障或抛出异常，Laravel的异常处理程序(我们在`app.php `中绑定的)将拾起异常并报告/呈现给用户。看看`sendrequestthroughRouter`，我们看到了一些很酷的东西 。*

```php
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

> First things first, we bind an **instance** of the Request class to the Container (we have only **one** request object per HTTP request) and clear already resolved instances of the Request class. Then we call this `bootstrap()` method that performs everything we were waiting for. Taking a look at that method, we see that we're essentially calling `$this->app->bootstrapWith($bootstrappers)` method on our Container, passing through list of bootstrappers from the `Kernel@$bootstrappers` property:
>
> *首先，我们将请求类的**实例**绑定到容器(每个HTTP请求只有**一个**请求对象)并清除请求类的已解析实例。然后我们调用这个`bootstrap()`方法来执行我们所等待的所有操作。看一下这个方法，我们看到我们实际上是在容器上调用`$this->app->bootstrapWith($bootstrappers)`方法，从`kernel@$ bootstrappers`属性中传递bootstrappers列表*

```php
protected $bootstrappers = [
    \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
    \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
    \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
    \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
    \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
    \Illuminate\Foundation\Bootstrap\BootProviders::class,
];
```

### Bootstrapping core components

`bootstrapWith` method just fires an event that we're bootstrapping a component, calls `bootstrap()` method on each component and fires an event that we have bootstrapped a component. I won't copy and examine code of all bootstrappers as their names clearly explain what they do, but I will try to summarize the code in this list. You can also take a look at the code of those components [here](https://github.com/laravel/framework/tree/5.6/src/Illuminate/Foundation/Bootstrap).

*调用bootstrapper的bootstrap方法引导各种功能*

- ```
  LoadEnvironmentVariables
  
```
  
 *读取环境变量*
  
literally creates a new instance of the
  
 
  
  ```php
  DotEnv
```
  
 
  
class and reads from
  
 
  
  ```
  .env
```
  
 
  
file
  
  - Also checks if configuration is cached, so it does not read `.env` again
- Also checks if we're in specific environment to read that `.env.{environment}` file
  
- ```
  LoadConfiguration
  ```

   *加载配置*

  checks the cached configuration and loads it (if it exists), otherwise loops through using Symfony's

   

  Finder

   

  component through all files in your config directory

  - It also sets the default timezone you specified in your `app.php` config: `date_default_timezone_set($config->get('app.timezone', 'UTC'));`
  - It also sets the default UTF-8 encoding: `mb_internal_encoding('UTF-8');`

- `HandleExceptions` uses PHP's error reporting functions to essentialy disable PHP error handling and setting up our custom error handler, so we can use libraries such as [Whoops!](https://github.com/filp/whoops) to show stack traces

- ```
  RegisterFacades
  ```

   *注册Facades*

  registers all the facades by storing their implementation into the Container. So when you call

   

  ```
  Facade::method()
  ```

  , Laravel will actually resolve that class out of the Container and call non-static

   

  ```
  method()
  ```

   

  on them

  - It also registers [real-time facades](https://laravel.com/docs/5.6/facades#real-time-facades)
  - It also uses Facades from packages

- ```
  RegisterProviders
  ```

  *注册config/app.php中的Providers*

  literally delegates to

   

  ```
  $app->registerConfiguredProviders();
  ```

  . This method loops through the

   

  ```
  providers
  ```

   

  array in the

   

  ```
  config/app.php
  ```

   

  file and registers those service providers into the Container

  - It also reads all package service providers and binds them
  - **Note: this is where contents of the `register()` method of your service providers are evaluated**

- ```
  BootProviders
  ```

   *引导注册过的service providers*

  literally delegates to

   

  ```
  $app->boot();
  ```

  , which loops through all registered service providers and calls

   

  ```
  boot()
  ```

   

  method on each.

  - **Note: this is where contents of the `boot()` method of your service providers are evaluated**

> **I believe you have now noticed the differences between `register()` and `boot()`. If you need some core framework services, wait until framework registers all service providers, then store your code in `boot()` method. If you need to get something done on provider registrations, put that code into `register()` method.**
>
> *我相信您已经注意到了`register()`和`boot()`之间的区别。如果您需要一些核心框架服务，请等到框架注册了所有服务提供者，然后将您的代码存储在`boot()`方法中。如果你需要完成提供商注册，把代码放到register()方法中*



> After this code has been evaluted, all that is left is send the request through the router and get back the response. Now, if you dump the contents of the Application class, we see that all our routes have been loaded, database connection has been instantied and we're ready to go. In this course, we will also take a look at some of the core components to see where exactly the connection is created, where the routes are loaded, etc. For now, this will be the end of part 2 and I hope you now know how your HTTP request is captured and how the framework is booted.
>
> *在对这段代码求值之后，剩下的就是通过路由器发送请求并获取响应。现在，如果您转储应用程序类的内容，我们将看到所有的路由都已加载，数据库连接已被即时连接，并且我们已经准备就绪。在本课程中，我们还将查看一些核心组件，以了解连接是在何处创建的，路由是在何处加载的，等等。现在，这将是第2部分的结尾，我希望您现在知道如何捕获HTTP请求以及如何启动框架。*

## Who wants to know more

As I said, we will take a look at the `make()` method on the Container. Well, `make()` [just delegates](https://github.com/illuminate/container/blob/master/Container.php#L599) to `resolve()` method. Taking a look at the `resolve()`, we see bunch of code (I have removed comments):

```php
protected function resolve($abstract, $parameters = [])
{
    $abstract = $this->getAlias($abstract);

    $needsContextualBuild = ! empty($parameters) || ! is_null(
        $this->getContextualConcrete($abstract)
    );

    if (isset($this->instances[$abstract]) && ! $needsContextualBuild) {
        return $this->instances[$abstract];
    }

    $this->with[] = $parameters;

    $concrete = $this->getConcrete($abstract);

    if ($this->isBuildable($concrete, $abstract)) {
        $object = $this->build($concrete);
    } else {
        $object = $this->make($concrete);
    }

    foreach ($this->getExtenders($abstract) as $extender) {
        $object = $extender($object, $this);
    }

    if ($this->isShared($abstract) && ! $needsContextualBuild) {
        $this->instances[$abstract] = $object;
    }

    $this->fireResolvingCallbacks($abstract, $object);

    $this->resolved[$abstract] = true;

    array_pop($this->with);

    return $object;
}
```

> As we see, we try to load the class name for the alias we provided (e.g. resolve `Illuminate\Container\Container` for `app` alias), then we check to see if we need [contextual binding](https://laravel.com/docs/5.6/container#contextual-binding). If we don't need contextual binding and we already have an instance of the class bound in the `$instances` property, just return that class instance. Otherwise, if we don't have an already bound instance, we should just build up the class and use [Reflection](http://php.net/manual/en/book.reflection.php) to resolve all dependencies. We see that we try to get concrete implementation of the alias (remember interface-binding) and see if the class is buildable (can be instantiated). If the class is buildable, we [call](https://github.com/illuminate/container/blob/master/Container.php#L758) `build()` method where we use Reflection to get the constructor of the class, get all of it's parameters and resolve them recursively as well. We see that if the constructor has no parameters, just `return new $class` and you're done. If we have dependencies, just recursively call `resolve()` on them! There is some additional checks to see if the class should be singleton (shared), then just resolve one instance and keep resolving that same class. Then we just fire some additional events, and we're done!
>
> *正如我们所看到的，我们尝试为我们提供的别名加载类名(例如，为`app`别名解析`Illuminate\Container\Container`)，然后检查是否需要[上下文绑定](https://laravel.com/docs/5/6/container #上下文绑定)。如果我们不需要上下文绑定，而且我们已经在`$instances`属性中有了类绑定的实例，那么只需返回该类实例。否则，如果我们没有一个已经绑定的实例，我们应该构建类并使用[Reflection](http://php.net/manual/en/book.reflection.php)来解决所有依赖项。我们尝试获得别名的具体实现(请记住接口绑定)，并查看类是否可构建(可以实例化)。如果类是可构建的，我们[调用](https://github.com/illuminate/container/blob/master/Container.php#L758)`build() `方法，我们使用反射来获取类的构造函数，获取类的所有参数，并递归地解析它们。我们看到，如果构造函数没有参数，只需`return new $class `，这样就完成了。如果我们有依赖，只需递归调用`resolve()`对他们!还有一些额外的检查，看看这个类是否应该是单例的(共享的)，然后解析一个实例并继续解析同一个类。然后我们只需触发一些额外的事件，就完成了!*