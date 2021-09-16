# Laravel 幕后：生命周期-响应

**本文翻译自 [LARAVEL BEHIND THE SCENES](https://crnkovic.me/tag/laravel-behind-the-scenes)** 

>  This is the final part of the Lifecycle series and we will take a look at how Laravel responds back to the user, whether that is the View response, JSON response or anything else. Let's start with part 4!
>
>  *这是生命周期系列的最后一部分，我们将查看Laravel如何响应用户，无论是视图响应、JSON响应还是其他响应。让我们从第4部分开始!*

**This guide covers the source code of version 5.6 of the Laravel framework.**

## Review

> In [part 1](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-container), we reviewed what happens as soon as `index.php` file loads up, even before core framework services load. In [part 2](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-boot-the-framework), we reviewed how the framework loads the configuration, sets up error handling, registers all service providers and resolves the facades and in [part 3](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-routing) we reviewed how Laravel points your request to the specific route and loads up the controller to handle that route. Now, let's move onto the how your controller response is automagically converted to proper response format and displayed to your browser. 
>
> *在第1部分中，我们回顾了`index.php`文件加载后会发生什么，甚至是在核心框架服务加载之前。在第2部分中，我们回顾了框架如何加载配置、设置错误处理、注册所有服务提供者和解析facades，在第3部分中，我们回顾了Laravel如何将您的请求指向特定路由并加载控制器来处理该路由。现在，让我们看看如何将控制器响应自动转换为适当的响应格式并显示到浏览器中。*

## Prepare the Response

> In part 3, we have seen that [`toResponse`](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php#L709) method converts any data you pass to it into a Response object. **We also noticed that `$response` variable is direct output of the controller.** 
>
> *在第3部分中，我们已经看到toResponse方法将传递给它的任何数据转换为响应对象。我们还注意到$response变量是控制器的直接输出。*

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
    } elseif (! $response instanceof SymfonyResponse && ($response instanceof Arrayable || $response instanceof Jsonable || $response instanceof ArrayObject || $response instanceof JsonSerializable || is_array($response))) {
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

> We realized that this is the place where Laravel magically converts arrays, strings and Eloquent models to JSON (because they implement the serializable interface and contain `toJson` method). If we now take a look at the Response class from the `Illuminate\Http` namespace, we see that this class inherit the identically titled class from the Symfony's HttpFoundation component. We also see that this class doesn't contain a lot of code. That's because most of the setup is done behind the scenes by Symfony. Before we take a look at the preparation method, creating a new instance of the class calls the constructor, right? So we should probably [take a look](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L194) at what it does.
>
> *我们意识到这是Laravel神奇地将数组、字符串和Eloquent models 转换成JSON的地方(因为它们实现了可序列化的接口并包含'`toJson `方法)。如果我们现在看一下来自`Illuminate\Http`名称空间的响应类，我们会看到这个类从Symfony的HttpFoundation组件继承了标题相同的类。我们还看到这个类不包含很多代码。这是因为大部分的设置都是在幕后由Symfony完成的。在查看准备方法之前，创建类的新实例调用构造函数，对吗?所以我们可能应该[看一看](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L194)。*

### New instance

> Right off the bat we see that we're creating new header bag from the empty `$headers` array, because no initial headers have been sent during the instantiation process (`new Response($response)`). 
>
> *我们马上看到我们正在从空的`$headers`数组中创建新的响应头，因为在实例化过程中没有发送初始头文件`(new Response($ Response))`。*

```php
public function __construct($content = '', int $status = 200, array $headers = array())
{
    $this->headers = new ResponseHeaderBag($headers);
    $this->setContent($content);
    $this->setStatusCode($status);
    $this->setProtocolVersion('1.0');
}
```

> If you examine the contents of the Response object after headers have been stored, you'll see that it's empty -- only date and cache-control headers have been set. Then, we set some initial properties like HTTP version, response content and status code and finally proceed with the preparation. If we `dd` this class after initial properties have been set, we see the following: (keep in mind that this is fresh Laravel installation that loads *welcome* view):
>
> *如果您在头信息存储后检查响应对象的内容，您将看到它是空的——只有date和cache-control头信息被设置。然后，我们设置一些初始属性，如HTTP版本、响应内容和状态代码，最后进行准备。如果我们'`dd()`这个类后，初始属性已被设置，我们看到以下:(记住，这是新鲜的Laravel安装加载*欢迎*视图):*

```php
Response{
	+headers:ResponseHeaderBag{}
	#content:"HTML output"
	version:"1.0"
	statusCode:200
	Original:View{
		path:"/resources/views/welocome.blade.php"
		view:"welcome"
		factory:Factory{}
	}
}
```

> We should now note couple of things: `original` property is a View object, `statusText` property sets the status text based on the HTTP code (404 - Not Found, 200 - OK, etc.) and `content` contains literal output that is going to be rendered to the client (HTML in this case). I'm kinda curious about this View object... Fun fact is that `setContent` method is [actually called from](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Http/Response.php#L25) the Laravel's Response class. Removing the comments and we see this method is pretty simple and clean. 
>
>  现在我们应该注意以下几点:`original`属性是一个视图对象，`statusText`属性根据HTTP代码设置状态文本(404 - Not Found, 200 - OK，等等)，`content`包含将呈现给客户端的文字输出(本例中为HTML)。我对这个视图对象有点好奇…有趣的是，`setContent`方法实际上是从Laravel的Response类调用的。删除注释，我们看到这个方法非常简单和干净。 

```php
public function setContent($content)
{
    $this->original = $content;

    if ($this->shouldBeJson($content)) {
        $this->header('Content-Type', 'application/json');

        $content = $this->morphToJson($content);
    } elseif ($content instanceof Renderable) {
        $content = $content->render();
    }

    parent::setContent($content);

    return $this;
}
```

> First things first, we save the initial content (object, view, etc.) into the `original` property in case we need it later. Then, we modify the `$content` variable however we need to. Have an object that should be returned as JSON (string, model, collection, JSON response, etc.)? Just set `Content-Type` header to `application/json` and [encode](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Http/Response.php#L71) the output to JSON. We returned a view? Or any object that implements `Renderable` interface? Call the `render` method on that object and call the `setContent` method from the parent on the modified content. In our case, `render` method compiles the view with Blade and creates raw HTML. In this moment, we can print out the `$content` and it will literally show the final HTML, but we're not done here.
>
> *首先，我们将初始内容(对象、视图等)保存到原始属性中，以备以后需要。然后，根据需要修改`$content`变量。是否有一个应该作为JSON返回的对象(字符串、模型、集合、JSON响应等)?只需将`Content-Type`头设置为`application/json`并将输出编码为json。我们返回了一个视图?或者任何实现`Renderable` 接口的对象?对该对象调用`render`方法，对修改后的内容从父对象调用`$setContent`方法。在我们的例子中，`render`方法使用Blade编译视图并创建原始HTML。此时，我们可以打印出`$content`，它将显示最终的HTML，但是这里还没有完成。*

### Prepare the headers

> Yep, now that we have created a new instance of the Response class in our Router, we are ready to call the `prepare` method. [As we can see](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L257) this method contains a lot of code, so I won't paste it here to not clutter the post. First, we should figure out if the response is informational or empty. The thing to know here is that response [is informational](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#Information_responses) if the status code is between **100** and **200**, and [is empty](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) if the status code is **204** or **304**. If it is informational or empty, remove any content and content-related headers. Otherwise set the content type and content length based on the Request also set the proper charset. **Also, default the HTTP version to the v1.1**. Finally, [check](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L1215) if Cache-Control should be removed for SSL encrypted downloads for IE. 
>
> *是的，现在我们已经在路由器中创建了Response类的一个新实例，我们准备调用prepare方法。正如我们所看到的，这个方法包含了很多代码，所以我不会把它粘贴到这里，以免影响文章。首先，我们应该确定响应是信息性的还是空的。这里需要知道的是，如果状态码在100和200之间，则响应是信息性的，如果状态码是204或304，则响应是空的。如果它是信息性的或空的，则删除任何与内容相关的标头。否则，根据请求设置内容类型和内容长度也要设置适当的字符集。同样，默认的HTTP版本是v1.1。最后，检查是否缓存控制应删除SSL加密下载的IE。*

### JSON & Redirect response

> JSON and Redirect responses are just slightly modified versions of the base Response object. [JSON response](https://github.com/laravel/framework/blob/972b82a67c6dd09fa01bf5e0b349a547ece33666/src/Illuminate/Http/JsonResponse.php) is represented by the Laravel's `JsonResponse` class which inherits the Symfony's class and performs some JSON validation, encoding options, sets the JSON content-type header, etc. [Redirect response](https://github.com/laravel/framework/blob/ba0fa733243fd9b5abac67e40c9a1f7ba2ca0391/src/Illuminate/Http/RedirectResponse.php) does exactly what it says and it also inherits the Symfony's class. The only thing this class does is sets the `Location` header to target URL. 
>
> *JSON和重定向响应只是基本响应对象的稍加修改的版本。JSON响应由Laravel的JsonResponse类表示，该类继承Symfony的类，并执行一些JSON验证、编码选项、设置JSON内容类型头等。重定向响应完全按照它说的去做，它还继承了Symfony的类。这个类唯一做的事情是将Location头设置为目标URL。*

#### Redirector-重定向器

> [Redirector](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Redirector.php) is a special class that wraps the redirect response behind some common functions: redirect back, redirect home, redirect to route, etc. So when you call the redirect facade/helper, it actually boots up the Redirector object. It also uses `URLGenerator` [class](https://github.com/laravel/framework/blob/30d2f7f350e1e5bfa742e18e993deedefec2e953/src/Illuminate/Routing/UrlGenerator.php) for generating URLs to named routes, previous URL from header/session, etc. So, for example, `redirect()->back()` will actually call this little piece of code:
>
> *重定向器是一个特殊的类，它将重定向响应封装在一些常用的函数后面:重定向返回、重定向home、重定向到route等。因此，当您调用重定向facade/helper时，它实际上会启动重定向对象。它还使用URLGenerator类来生成指向指定路由的URL、来自标头/会话的前一个URL，等等。例如，redirect()->back()实际上会调用这一小段代码:*

```php
public function back($status = 302, $headers = [], $fallback = false)
{
    return $this->createRedirect($this->generator->previous($fallback), $status, $headers);
}
```

> If you take a look at the `createRedirect` method, this method literally just creates a new instance of the `RedirectResponse` class and sets the session and the request in the Response.
>
> *如果您查看一下“createRedirect”方法，这个方法实际上只是创建一个“RedirectResponse”类的新实例，并在响应中设置会话和请求。*

#### Response helper-Response 类的助手函数

```php
// Illuminate/Foundation/helpers.php
function response($content = '', $status = 200, array $headers = [])
{
    $factory = app(ResponseFactory::class);

    if (func_num_args() === 0) {
        return $factory;
    }

    return $factory->make($content, $status, $headers);
}
```

## Finish the lifecycle

> We have now done every step through the lifecycle of Laravel application and reverted back through the stack to that famous `index.php` file. The only thing left to do here to display (send) the response to the client and call any terminable middleware.
>
> *我们现在已经完成了Laravel应用程序生命周期的每一步，并通过堆栈返回到那个著名的index.php文件。这里剩下要做的惟一一件事就是显示(发送)对客户机的响应并调用任何可终止的中间件。*

```php
$response->send();

$kernel->terminate($request, $response);
```

### Send the response

> We prepared headers, compiled view into the HTML, done all business logic in our controllers and models and the only thing left to do is to send headers and show the content. [This is what the `send` method of the Symfony's Response class does](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L356).
>
>  *我们准备了标题，将视图编译成HTML，在控制器和模型中完成了所有的业务逻辑，剩下要做的就是发送标题和显示内容。[这就是Symfony的Response类的' send '方法的作用](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L356)。* 

```php
public function send()
{
    $this->sendHeaders();
    $this->sendContent();

    if (function_exists('fastcgi_finish_request')) {
        fastcgi_finish_request();
    } elseif (!\in_array(PHP_SAPI, array('cli', 'phpdbg'), true)) {
        static::closeOutputBuffers(0, true);
    }

    return $this;
}
```

> It's worth noting that this method also does some additional checks to flush the data to the client if server is using [PHP-FPM](http://php.net/manual/en/function.fastcgi-finish-request.php). *If you're using Laravel Valet and have PHP installed through brew, this function will most certainly be called*. [There is also a method to close output buffer](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L1193) and flush the content if using output control, but we won't cover it under this guide. If you're interested, take a look at [the docs](http://php.net/manual/en/book.outcontrol.php).
>
>  *值得注意的是，如果服务器使用[PHP-FPM](http://php.net/manual/en/function.fastcgi-finishrequest.php)，此方法还会执行一些额外的检查，将数据刷新到客户端。*如果您正在使用Laravel Valet，并通过brew安装了PHP，这个函数肯定会被调用*。[还有一个方法来关闭输出缓冲区](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/Response.php#L1193)和刷新内容，如果使用输出控件，但我们不会在本指南中涵盖它。如果您感兴趣，可以看看[文档](http://php.net/manual/en/book.outcontrol.php)。*

#### Send the headers

> Taking a peek at the `sendHeaders` method, we see that we're using two native PHP functions: `headers_sent` and `header`.
>
> *看一下`sendHeaders`方法，我们看到我们使用了两个原生PHP函数:`headers_sent`和'`header `。*

```php
public function sendHeaders()
{
    if (headers_sent()) {
        return $this;
    }

    foreach ($this->headers->allPreserveCase() as $name => $values) {
        foreach ($values as $value) {
            header($name.': '.$value, false, $this->statusCode);
        }
    }

    header(sprintf('HTTP/%s %s %s', $this->version, $this->statusCode, $this->statusText), true, $this->statusCode);

    return $this;
}
```

> Remember those old errors that occurred if you messed up your CodeIgniter app -- *Headers have already been sent*? Well, [conditional `headers_sent` prevents that](http://php.net/manual/en/function.headers-sent.php). If we have not already sent headers, we should send them now! Looping through the `headers` property and calling the [PHP's `header` method](http://php.net/manual/en/function.header.php) on each item does exactly that. Also, we need one additional header call to set the status code and the version of the HTTP protocol. This step is done out of the loop because all other headers have a shape of *"Name: Header"* (e.g. `header('Content-Type: application/json');`), while status header looks like *"HTTP/version code text"*, e.g. `header("HTTP/1.0 404 Not Found");`
>
> *还记得你搞砸CodeIgniter应用程序时发生的那些老错误吗?条件`headers_sent`可以防止这种情况。如果我们还没有发送头文件，我们应该现在就发送!循环遍历`headers`属性并在每个项目上调用PHP的`header`方法就是这样做的。此外，我们还需要一个额外的头调用来设置状态代码和HTTP协议的版本。这一步是在循环之外完成的，因为所有其他头的格式都是“Name: Header”(例如Header ('Content-Type: application/json');)，而状态头看起来像“HTTP/版本代码文本”，例如`header(“HTTP/1.0 404 Not Found”);`*

#### Send the content

> After sending the headers, we need to send the content to the user. **Remember when we said we can literally just echo out the content and get the HTML response?** This is exactly what the `sendContent` method does. It's just an [`echo`](http://php.net/manual/en/function.echo.php) call:
>
> *在发送标题之后，我们需要将内容发送给用户。**还记得我们说过我们可以直接回显内容并得到HTML响应吗?**这正是' sendContent '方法所做的。它只是一个[` echo `](http://php.net/manual/en/function.echo.php)调用:*

```php
public function sendContent()
{
    echo $this->content;

    return $this;
}
```

As of this moment, the response is sent and the browser displays the contents to the client. Or if you're using APIs, the JSON is responded!

### Terminate the application

> After sending the response, the only step in the lifecycle that needs to be done is to call any additional *terminable* middleware. We see that `terminate` method on the Kernel in `index.php` is called -- `$kernel->terminate($request, $response);`, and that method delegates `terminate` method on the Application instance but not before it calls any additional middleware.
>
>  在发送响应之后，生命周期中惟一需要完成的步骤是调用任何附加的“可终止的”中间件。我们看到在`index.php`中的Kernel的`terminate`方法被调用`$kernel->terminate($request, $response);`，该方法将`terminate `方法委托给应用程序实例，但在它调用任何其他中间件之前不会这样做。 

```php
public function terminate($request, $response)
{
    $this->terminateMiddleware($request, $response);

    $this->app->terminate();
}
```

Now that we know how middleware works, we can just take a quick look at the `terminateMiddleware` method and figure out what it's doing:

```php
protected function terminateMiddleware($request, $response)
{
    $middlewares = $this->app->shouldSkipMiddleware() ? [] : array_merge(
        $this->gatherRouteMiddleware($request),
        $this->middleware
    );

    foreach ($middlewares as $middleware) {
        if (! is_string($middleware)) {
            continue;
        }

        list($name) = $this->parseMiddleware($middleware);

        $instance = $this->app->make($name);

        if (method_exists($instance, 'terminate')) {
            $instance->terminate($request, $response);
        }
    }
}
```

> After getting a list of middleware, loop through them and call the `terminate` method on the Kernel -- but only if one exists. Then, delegate to `terminate` method on the Application object which fires terminating event and calls any additional listeners that might have been created for that event. And with that, finally, the lifecycle of the entire application is done!
>
> *在获得中间件列表之后，循环遍历它们并在内核上调用`terminate`方法——但只有在存在这种方法时才这样做。然后，委托应用程序对象上的`terminate`方法，该方法触发终止事件并调用可能为该事件创建的任何其他侦听器。最后，完成了整个应用程序的生命周期!*

## Summary

> To summarize: we took a look at the first file that loads up when you make a request, how it registers core Laravel components, how it creates a HTTP kernel that registers and boots all service providers, creates the global request object from environment variables, runs the router and how that router executes your own code you provided in your controller method and finally responds back the response content to the client that made a request.
>
> *总结:当你提出请求时,它如何注册核心的Laravel组件,它如何创建一个注册和引导所有服务提供者的HTTP内核,创建来自环境变量的全局请求对象,运行路由器,以及路由器如何在控制器中执行你自己的代码,并最终对请求的客户端响应响应内容。*

------

