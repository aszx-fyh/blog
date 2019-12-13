# Interfaces-接口

> Digging through the Laravel's source code, I came across this little piece of code that you've probably seen before
>
> *通过挖掘Laravel的源代码，我发现了这一小段代码，您可能以前见过*

```php
abstract class Model implements ArrayAccess,Arrayable,Jsonable,JsonSerializable,QueueableEntity,UrlRoutable
```

> That's a lot of interfaces and I thought this may be a good topic since some of these interfaces are implemented in various classes throughout the entire framework and for a good reason too. 
>
> *有很多接口，我认为这可能是一个很好的主题，因为这些接口中的一些是在整个框架的各个类中实现的，这也是有原因的。*

## Why?

> You probably noticed that Laravel provides flexibility and customization everywhere. For example, you can create a JSON response out of your models, encode strings and arrays into the JSON, render emails in your browser. This is the point of these interfaces. Laravel can't know which classes you're using and what exactly is happening in your logic. That's why it's using interfaces to identify a class type of the object. Whether that object is the response from the controller, a view data or anything else, Laravel can dictate what it does. Example: let's say some URI is hit you return a simple object. How does Laravel know that this class should encode the object and output the JSON? How does Laravel know when you return a view, that it should render? The thing Laravel knows is that some object implemented for example Jsonable interface and knows that this object has the `toJson` method. Same thing for rendering, creating a response, etc.
>
> *您可能已经注意到Laravel到处都提供了灵活性和定制。例如，您可以从模型中创建JSON响应，将字符串和数组编码到JSON中，在浏览器中呈现电子邮件。这就是这些接口的意义所在。Laravel无法知道您正在使用哪些类，以及逻辑中究竟发生了什么。这就是为什么它使用接口来标识对象的类类型。无论该对象是来自控制器、视图数据或其他任何东西的响应，Laravel都可以指示它做什么。例如:假设某个URI被命中，你返回一个简单的对象。Laravel如何知道这个类应该对对象进行编码并输出JSON?Laravel如何知道当你返回一个视图时，它应该呈现?Laravel知道的是某个对象实现了Jsonable接口，并且知道这个对象有`toJson`方法。渲染、创建响应等也是如此。*

## Interfaces

各种具体的接口

### ArrayAccess -php原生提供

> We'll kick things of with this guy. **Note that this is native PHP interface and has nothing to do with Laravel, but Laravel uses this a lot.** As the [documentation](http://php.net/manual/en/class.arrayaccess.php) states, this is:
>
>  我们将和这个家伙打交道。注意，这是一个本机PHP接口，与Laravel无关，但Laravel经常使用它。 

> Interface to provide accessing objects as arrays.
>
> 提供像数组一样访问对象的接口。

> There's not much to talk about, if the `$config` variable holds an object, like [here](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Config/Repository.php), you can extend it's functionality with array-like access: `$config['key']` by implementing the `ArrayAccess` methods, like [Config/Repository.php has](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Config/Repository.php#L135). Laravel uses this in a [lot](https://github.com/laravel/framework/search?q=arrayaccess&unscoped_q=arrayaccess) of places: Eloquent, Collections, Container, Config repository, Cache repository, ...
>
> *如果`$config`变量包含一个对象，例如here，那么讨论不多。您可以通过实现数组访问方法扩展类似于数组访问的功能：`$config ['key'],`（像 ` Config/Repository.php`）。 Laravel在很多地方使用它:Eloquent, Collections, Container, Config repository, Cache repository*

Example:

```php
class User implements ArrayAccess
{
    protected $firstName = 'John';
    protected $lastName = 'Doe';

    public function offsetExists($key)
    {
        return property_exists($this, $key);
    }

    public function offsetGet($key)
    {
        return $this->{$key};
    }

    public function offsetSet($key, $value)
    {
        $this->{$key} = $value;
    }

    public function offsetUnset($key)
    {
        $this->{$key} = null;
    }
}

Route::get('/', function () {
    $user = new User;

    dd($user['firstName']);
});
```

### JsonSerializable

This is another interface that PHP provides and as the [documentation](http://php.net/manual/en/class.jsonserializable.php) says:

> Objects implementing JsonSerializable can customize their JSON representation when encoded with json_encode().
>
> 实现了`JsonSerializable `的对象可以自定义json表示，当它们被`json_encode()`编码的时候

Essentially you can modify how your object looks like when it's being encoded to JSON. If we take a simple User class for example:

```php
class User
{
    public $firstName = 'John';
    public $lastName = 'Doe';
}
```

> Once you try to encode this class with `json_encode(new User)` and dump the contents you'll get this: `"{"firstName":"John","lastName":"Doe"}"`. If you want to modify how the JSON representation looks like, implement the `JsonSerializable` interface located in the global namespace and implement the `jsonSerialize` method, for example:
>
> 实现JsonSerializable接口的jsonSerialize方法，用于展现json格式

```php
class User implements JsonSerializable
{
    public $firstName = 'John';
    public $lastName = 'Doe';

    public function jsonSerialize()
    {
        return ['name' => sprintf('%s %s', $this->firstName, $this->lastName)];
    }
}
```

> If we now dump the encoded object, we see exactly what we expected: `"{"name":"John Doe"}"`. This is also used throughout the [framework](https://github.com/laravel/framework/search?q=jsonserializable&unscoped_q=jsonserializable). For example, Eloquent model serializes the array representation of the model. This is used with Jsonable interface to provide flexibility when encoding objects to JSON, for Responses, etc.
>
> *如果我们现在转储编码的对象，我们将看到我们所期望的:“{”name“:”John Doe“}””。整个[框架](https://github.com/laravel/framework/search?例如，雄辩模型序列化模型的数组表示。它与Jsonable接口一起使用，以便在将对象编码为JSON、响应等时提供灵活性。*

### [Jsonable](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Contracts/Support/Jsonable.php)

> This is Laravel's interface and it serves a very similiar purpose as `JsonSerializable`, however with one small modification. Let's say we want to hook into `jsonSerialize` method and do some checking before encoding to the JSON. This is the purpose of `Jsonable` interface -- it dictates **how the object should be encoded, while JsonSerializable dictates how the encoded object looks like**. This interface implements `toJson($options = 0)` method. Example:
>
>  这是Laravel的接口，它的用途与`JsonSerializable`非常相似，只是做了一点小小的修改。假设我们想要挂钩到`jsonSerialize`方法，并在对JSON编码之前做一些检查。这就是`Jsonable`接口的用途——它规定了应该如何对对象进行编码，而JsonSerializable规定了编码后的对象的格式。这个接口实现了`toJson($options = 0) `方法。例子: 

```php
class User implements Jsonable
{
    public $firstName = 'John';
    public $lastName = 'Doe';

    public function toJson($options = 0)
    {
        if ($this->firstName === 'John') {
            throw new Exception('Name must not be John.');
        }

        return json_encode(['name' => sprintf('%s %s', $this->firstName, $this->lastName)], $options);
    }
}

Route::get('/', function () {
    return new User;
});
```

> If the name is John, it throws an exception, however if the name is not John, sure enough it encodes the JSON. **Note that you need to encode it to JSON yourself.** You can use this combined with `JsonSerializable` to not repeat yourself:
>
> 组合使用Jsonable和JsonSerializable

```php
class User implements Jsonable, JsonSerializable
{
    public $firstName = 'Jane';
    public $lastName = 'Doe';

    public function toJson($options = 0)
    {
        if ($this->firstName === 'John') {
            throw new Exception('Name must not be John.');
        }
        return json_encode($this->jsonSerialize(), $options);
    }

    public function jsonSerialize()
    {
        return ['name' => sprintf('%s %s', $this->firstName, $this->lastName)];
    }
}
```

### [Arrayable](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Contracts/Support/Arrayable.php)

> This is essentially the same concept as Jsonable, but it dictates how the object looks when represented as an array. And this is where the infamous `toArray` method comes from. Some of the examples are Collections and view data. Collections operate on a collection of data, but Collection class shouldn't care if your object is of type User, or Car, or Pet, or whatever. As long as your classes implement this interface, we know that this object can be represented as an array. In views, once you pass an object as view data, and that object implements the Arrayable interface, it calls the `toArray` method on the object as can be seen [here](https://github.com/laravel/framework/blob/fe8571be14566e853408b416798f5f39fa976227/src/Illuminate/View/Factory.php#L233).
>
>  *这本质上与Jsonable是相同的概念，但它规定了对象在表示为数组时的外观。这就是臭名昭著的`toArray`方法的由来。其中一些示例是集合和视图数据。集合对数据集合进行操作，但是集合类不应该关心对象的类型是User、Car、Pet还是其他类型。只要您的类实现了这个接口，我们就知道这个对象可以表示为一个数组。在视图中，一旦您将一个对象作为视图数据传递，并且该对象实现了Arrayable接口，它就会调用对象上的`toArray`方法。*

Example:

```php
class User implements Arrayable
{
    protected $firstName = 'John';
    protected $lastName = 'Doe';

    public function toArray()
    {
        return ['name' => sprintf('%s %s', $this->firstName, $this->lastName)];
    }
}

Route::get('/', function () {
    return view('welcome', new User);
});
// view:
{{ $name }}
```

### [Responsable](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Contracts/Support/Responsable.php)

> This interface requires the implementation of `toResponse` method and I addressed it in my [Lifecycle: Response](https://crnkovic.me/laravel-behind-the-scenes-lifecycle-response/) article. It's not so widely used, but it can help you out a lot because you can format the response however you want. Any class is *responsable* as long as it implements this interface. As you can see [here](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Routing/Router.php#L709), Laravel checks to see if the response is the instance of Responsable and invokes the `toResponse` method. Full power of this interface was briefly talked about at Laracon EU 2017, but Taylor mentioned it probably won't make it into the 5.5... and it did not. How you can use this interface is up to you, but combined with API resources, and it's really powerful.
>
>  这个接口需要实现“toResponse”方法，我在我的[Lifecycle: Response](https://crnkovic.me/laravel-幕后-lifecycle-response/)文章中解决了这个问题。它并没有被广泛使用，但是它可以帮助您解决很多问题，因为您可以根据自己的需要格式化响应。任何类都是可响应的，只要它实现了这个接口。正如您所看到的， Laravel检查响应是否是Responsable的实例，并调用`toResponse`方法。2017年Laracon EU大会上曾简要讨论过该接口的全部功能，但泰勒表示，该接口可能无法进入5.5……但事实并非如此。如何使用这个接口由您自己决定，但是要结合API资源，它非常强大。 

> Let me give you a simple example: So the core life cycle is request-response, right? You can structure your controllers in such a way. Where Request is an object, Response is an object and perform some business logic in between. So any component is reusable again.
>
>  让我给你一个简单的例子:因此核心生命周期是请求-响应，对吗?你可以这样构造你的控制器。其中Request是对象，Response是对象，并在两者之间执行一些业务逻辑。所以任何组件都是可重用的。 

```php
class Responses\AvatarUploaded implements Responsable
{
    protected $success;

    public function __construct($success)
    {
        $this->success = $success;
    }

    public function toResponse($request)
    {
        if (!$this->success) {
            return new JsonResponse('Something went wrong!', 400);
        }

        return new JsonResponse('Your avatar has been uploaded.');
    }
}

// UploadAvatar performs a validation
Route::post('/avatar', function (Requests\UploadAvatar $request) {
    $avatar = new Services\UserAvatar($request->user());

    // generate thumbnails, remove old avatar, etc...
    $success = $avatar->upload($request->avatar);

    return new Responses\AvatarUploaded($success);
});
```

Your controllers will look really clean!

### [Htmlable](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Contracts/Support/Htmlable.php)

> This less known interface (that looks like somebody smashes the keyboard) is also not so widely used, but it's really powerful. It only requires the implementation of one method: `toHtml`. As you may know, Blade tries to escape everything between the curly brackets before rendering in the browser, as can be seen [here](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/View/Compilers/Concerns/CompilesEchos.php#L89). However, the escape helper checks if the content between double curly brackets is an instance of Htmlable then invokes the implemented `toHtml` method.
>
>  这个不太为人所知的接口(看起来就像有人打碎了键盘)也没有被广泛使用，但它确实很强大。它只需要实现一个方法:`toHtml `。您可能知道，在浏览器中呈现之前，Blade尝试在花括号之间转义所有内容，如下所示。但是，escape助手检查双花括号之间的内容是否是Htmlable的实例，然后调用实现的`toHtml`方法。 

```php
function escape($value, $doubleEncode = true)
{
    if ($value instanceof Htmlable) {
        return $value->toHtml();
    }

    return htmlspecialchars($value, ENT_QUOTES, 'UTF-8', $doubleEncode);
}
```

> So in essence, any object can be rendered in Blade with `{{ }}` as long as it implements this interface. You can use it in conjunction with [HtmlString](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Support/HtmlString.php) class to get the most out of it. The best example I can think of for Htmlable interface is rendering pagination links with `@{{ $collection->links() }}`. The paginator's `links` method delegates the `render()` method which creates a new instance of HtmlString, which is in fact Htmlable!
>
>  因此，本质上，任何对象都可以在Blade中使用‘{{}}’来呈现，只要它实现了这个接口。您可以将它与[HtmlString](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Support/HtmlString.php)类结合使用，以获得最大的好处。对于Htmlable接口，我能想到的最好的例子是使用`@{{$collection->links()}}`来呈现分页链接。paginator的`links`方法委托了` render()`方法，该方法创建一个新的HtmlString实例，实际上它是Htmlable! 

### [Renderable](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Contracts/Support/Renderable.php)

> Surprise, surprise -- anything that is renderable should implement this. Not much to talk about, currently only View and Mail objects implement this, but anything else can implement this interface. As you can see [here](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Http/Response.php#L41), Laravel checks to see if your object is Renderable and calls the `render()` method on your object to render the data.
>
>  惊喜，惊喜——任何可呈现的东西都应该实现这一点。没有太多要讨论的，目前只有视图和邮件对象实现这个接口，但是其他任何东西都可以实现这个接口。正如你所看到的， Laravel检查你的对象是否可渲染，并调用对象的`render()`方法来渲染数据。 

Example:

```php
class User implements Renderable
{
    protected $firstName = 'John';
    protected $lastName = 'Doe';

    public function render()
    {
        return "<h1>{$this->firstName} {$this->lastName}</h1>";
    }
}

Route::get('/', function () {
    return new User;
});
```

And sure enough, we get the HTML: `John Doe`. There's not much there, but essentially, this is it!

### [UrlRoutable](https://github.com/laravel/framework/blob/56a58e0fa3d845bb992d7c64ac9bb6d0c24b745a/src/Illuminate/Contracts/Routing/UrlRoutable.php)

> This one is a bit tricky to explain. It's a contract that makes any object routable. Meaning what? It's best to see it in action:
>
>  这一点很难解释。它是使任何对象可路由的契约。意思什么?最好是它的行为: 

```php
class Users
{
    public function find($key, $value)
    {
        $users = [
            new User(1, 'John', 'Doe'),
            new User(2, 'Jane', 'Doe'),
            new User(3, 'Jeff', 'Doe')
        ];

        return collect($users)
            ->where($key, $value)
            ->first();
    }
}

class User implements UrlRoutable, JsonSerializable
{
    public $id;
    public $firstName;
    public $lastName;

    public function __construct($id = null, $firstName = null, $lastName = null)
    {
        $this->id = $id;
        $this->firstName = $firstName;
        $this->lastName = $lastName;
    }

    public function getRouteKey()
    {
        return $this->{$this->getRouteKeyName()};
    }

    public function getRouteKeyName()
    {
        return 'id';
    }

    public function resolveRouteBinding($value)
    {
        return (new Users)->find($this->getRouteKeyName(), $value);
    }

    public function jsonSerialize()
    {
        return [
            'name' => sprintf('%s %s', $this->firstName, $this->lastName)
        ];
    }
}

Route::get('{user}', function (User $user) {
    return $user;
})->name('home');
```

> Let's say we have a list of users in a class called Users. Then we implemented the UrlRoutable interface in the User class that allows us to typehint User object in routes and use route model binding. When we hit the URI, Laravel goes into the `resolveRouteBinding()` method and passes the parameter from the URI. `getRouteKeyName` is a method that indicates on which property should be check for a resource. And finally, if we want to generate a URL for our object without explicitly having to specify the key: `route('home', ['id' => 1])`, we can leverage what will be passed on to the route parameter with `getRouteKey` method. In this case, we can see that we're referencing 'id' property on `$this`.
>
>  假设在一个名为Users的类中有一个用户列表。然后，我们在User类中实现了UrlRoutable接口，该接口允许我们在路由中键入User对象并使用路由模型绑定。当我们到达URI时，Laravel进入`resolveRouteBinding()`方法，并从URI传递参数。`getRouteKeyName`是一个方法，它指示应该检查哪个属性上的资源。最后，如果我们想为对象生成一个URL，而不需要显式地指定键:`route('home'， ['id' => 1])`，我们可以使用`getRouteKey`方法来利用传递给route参数的内容。在本例中，我们可以看到我们在`$this`上引用了'id'属性。

Example:

```php
$user = (new Users)->find('id', 2);

dd(route('home', $user));
```

> This generates the URI for our route: `domain.com/2`. Even though I'm not really sure about use-case for this interface, but it's nice to know how it works.
>
> 这为我们的路线生成URI: `domain.com/2`。尽管我不是很确定这个接口的用例，但是很高兴知道它是如何工作的。