# Supports-辅助(支持)

> Usually, your project will contain some code that needs to be reused throughout the codebase. So does the Laravel framework itself. And it's a good practice to extract that code into reusable classes. Think of arrays and "dot" notation. Laravel allows you to get certain value from a nested array using "dot" notation in various places: configs, request params, translations, etc. So Laravel provides us with an `Arr` class that holds common logic for arrays, such as data retrieval using "dot" notation, etc. Enter: support components. Support components are various classes or traits whose logic is being reused throughout the framework. There are various support classes, and we'll talk about few: Fluent class, Manager, something we call macros, string and array classes, Collection and helpers. We'll go over each of those and explain what they're purpose is, where they're used and we'll show a use-case for each of those. All of those are placed within `Illuminate\Support` namespace. **Note that all of those classes/traits can be used in your app and is not limited to the framework itself.**
>
> *通常，您的项目将包含一些需要在整个代码库中重用的代码。Laravel框架本身也是如此。将这些代码提取到可重用的类中是一个很好的实践。想想数组和“点”符号。Laravel允许您在不同的地方使用“点”符号从嵌套的数组中获取特定的值:配置、请求参数、翻译等等。所以Laravel为我们提供了一个`Arr`类，它包含数组的公共逻辑，比如使用“点”表示法进行数据检索，等等。输入:支持组件。支持组件是各种类或特征，它们的逻辑在整个框架中被重用。有各种各样的支持类，我们只讨论几个:Fluent类、管理器、一些我们称为宏的东西、字符串和数组类、集合和助手。我们会详细讨论每一个，并解释它们的用途是什么，它们在哪里被使用，我们会展示每一个的用例。所有这些都放在`Illuminate\Support`名称空间中。注意，所有这些类/特征都可以在你的应用中使用，并不局限于框架本身*
>
> ## Macros
>
> If you're familiar with Swift's Extensions, the Macros are essentially the same thing. Macros are a way to hook into the framework and extend with some additional functionality without explicitly inheriting the class. A lot of Illuminate components use the Macroable trait which allows the class to be extended. If we dig into the code and take a look at that trait, you can see a static property `$macros`. The effect of that static property is **global access**. If you assign a macro to Collection class, all Collection instances will be able to access the same macro.
>
> *如果你熟悉Swift的扩展，那么这些宏本质上是一样的。宏是一种连接到框架并使用一些附加功能进行扩展的方法，而不需要显式地继承类。许多`Illuminate components`使用了允许类扩展的宏特征。如果我们深入研究代码并查看该特征，您可以看到一个静态属性`$macros`。该静态属性的作用是**全局访问**。如果将宏分配给集合类，则所有集合实例都能够访问相同的宏。*
>
> ### Usage
>
> #### Registering the macro
>
> Binding the macro into the class is done by calling the `macro($name, $macro)` static method. Method literally binds the `$macro` (Closure that performs your functionality) into `$macros` array by the key `$name`. Using this, you can extend, for example, the Collection class like it's described in the docs. Note that this is usually done in a service provider.
>
> *通过调用`macro($name， $macro)`静态方法将宏绑定到类中。方法通过键`$name`将`$macro`(执行功能的闭包)绑定到`$macros`数组中。使用它，您可以扩展，例如，像文档中描述的那样扩展集合类。注意，这通常是在服务提供者中完成的。*
>
> #### Accessing the macro
>
> We can also see the `__call` and `__callStatic` magic methods. Those methods are accessed whenever there is a call on inaccessible method on a class. We want to override that to see if the macro exists (`static::hasMacro($macro)`) and if it does, resolve functionality from the array and invoke it. A functionality is usually a Closure, but can be anything callable. Note that you can macro in another object if you'd like by calling the mixin method instead. In case you haven't noticed, calling `$this` inside a Closure for a macro will access the current instance of the class itself. How can this be possible? How can I use `$this` to access an object's properties and methods inside a Closure? Well, Closures in PHP have this method called bindTo and Laravel takes advantage of that. It binds the current object being macro'ed to the Closure, so `$this` references the class itself, not the Closure.
>
> *我们还可以看到“调用”和“调用静态”的魔法方法。只要在类上调用了不可访问的方法，就会访问这些方法。我们想要覆盖它来查看宏是否存在(`static::hasMacro($macro)`)，如果存在，则从数组中解析功能并调用它。功能通常是一个闭包，但可以是任何可调用的东西。请注意，如果您愿意，可以通过调用mixin方法在另一个对象中进行宏。如果您没有注意到，在宏的闭包中调用`$this `将访问类本身的当前实例。这怎么可能呢?如何使用`$this `在闭包中访问对象的属性和方法?PHP中的闭包有一个名为`bindTo`的方法，Laravel利用了这个方法。它将当前的对象宏绑定到闭包，所以`$this`引用的是类本身，而不是闭包。*
>
> ### Conflict
>
> If the class that used the Macroable trait has a `__call` method, there is going to be a conflict. Both methods can't be invoked, so when we use the trait, we should rename the Macroable's `__call` method into something else, but then make sure to run that method inside objects `__call` method. The example can be seen in the `Cache\Repository` class. We import the trait, but we also rename its `__call` into `macroCall`. Then, in the Repository's `__call`, we make sure to call the macroCall if the class has a macro, otherwise default to local method call — there is no conflict.
>
> *如果使用Macroable trait的类有一个`__call`方法，就会有冲突。这两个方法都不能被调用，所以当我们使用trait的时候，我们应该把Macroable的`__call`方法重命名为其他的东西，然后确保在对象的`__call`方法中运行这个方法。这个例子可以在`Cache\Repository`类中看到。我们导入这个trait，但是我们也把它的`__call`重命名为`macroCall`。然后，在仓库的`__call`中，如果类有宏，我们确保调用宏调用，否则默认调用本地方法——没有冲突。*
>
> **Note:** don't over-use it. If you have a lot of macros, or some really complex logic, it's much better to extend the class and do your logic than to fill up your service providers with macros.
>
> *注意:不要过度使用。如果您有很多宏，或者一些非常复杂的逻辑，那么最好扩展类并执行您的逻辑，而不是用宏填充服务提供者。*
>
> One macro I often like to use is checked for the Request class to determine whether or not a checkbox in a form is checked or not:
>
> *我经常使用的一个宏是检查请求类，以确定是否选中了表单中的复选框:*
>
> ```php
> Request::macro('checked', function ($attribute) {
>  return in_array($this->input($attribute, false), [true, 1, '1', 'on']);
> });
> ```
>
> ## Fluent
>
> I just recently discovered this and it blew me away. Okay, so what is it? You know when you store some array of items in your app and have to build all this boilerplate. Let's say you have website settings that you store as a key-value pairs in your database. You probably know that you had to build a class that has a getter and a setter method, where getter probably needs to return some default value in case the requested item doesn't exist and your app shouldn't blow up by trying to access an array value that doesn't exist but rather return `null`, for example. Then, you need to probably implement various interfaces since you want to access those items with array notation, such as `$settings['timezone']`, or in object notation, `$settings->timezone`. Then, you need to specify how it renders when displaying as JSON, etc. Yeah, this is some common boilerplate you would need to implement. Not anymore. `Illuminate\Support\Fluent` provides that boilerplate and the code can be seen here. If you have something like Settings class, you can inherit the Fluent class and add some additional methods such as save, or persist or something similar that updates the database, and override the constructor to load the items from the database and store into the object and very quickly you built the primitive key-value Settings store.
>
> *我最近才发现这一点，这让我大吃一惊。好吧，那是什么?你知道当你在你的应用程序中存储一些项目的数组时，你必须构建所有这些样板文件。假设您有一个网站设置，您将其存储为数据库中的键-值对。你可能知道你必须建立一个类,它有一个getter和setter方法,在getter可能需要返回一些默认值的情况下,要求项目不存在和应用程序不能炸毁试图访问数组值不存在返回‘空’,而是为`null`。然后，您可能需要实现各种接口，因为您希望使用数组表示法来访问那些项，比如`$settings['timezone'] `，或者在对象表示法中，` $settings->timezone `。然后，您需要指定它在显示为JSON时的呈现方式，等等。是的，这是一些需要实现的常见样板文件。不了,`lighting \Support\Fluent`提供了样板文件和代码。类如果你有类似的设置类,你可以继承Fluent类并添加一些额外的方法,如保存,或持续或类似的东西,更新数据库,并覆盖构造函数从数据库加载项并存储到对象,并很快你建立原始存储键值设置。*
>
> ## Manager
>
> A manager is a class that dictates which driver is currently being used for certain functionality of your app. It's best to show in an example. Think of caching: there are multiple cache stores you can use, such as memcached, redis, file, database, ... Cache manager says to Laravel: okay, you'll use the redis driver now, but if user says to use file driver, use that one. There are a lot of Illuminate components that work on drivers: sessions, authentication, queues, etc. And you can use the same pattern in your app. Let me provide you with an example, of the top of my head: let's say you are ranking some user content. You would create a class called Ranker that extends `Illuminate\Support\Manager`. You have multiple ranking algorithms implemented in your app, and each of those algorithm implementations are so-called drivers and you can swap them as you wish. All of them have the same methods dictated by some RankingAlgorithm interface. One day you want to use one algorithm, then if this one stops working for you for some reason, swap for another, etc. There are probably better examples, but oh well. There's a `Illuminate\Support\Manager` class that allows for this driver-based implementation. As you can see, if you're extending this class, you need to implement `getDefaultDriver` method to determine which driver is going to be used by default. In Cache case, this is read from the config file to determine which cache store the app will use. Then, once you call a method on your Ranker class, like rankContent, if the default driver is AlgorithmA, the Ranker will delegate to the rankContent method from the currently active driver. Now when you want to change for another driver, change the `getDefaultDriver` method to use AlgorithmB driver and you're done. Ranker class does not care which driver is being used and nothing else needs to be changed.
>
> *管理器是一个类，它指示应用程序的某些功能当前使用的是哪个驱动程序。想想缓存:有多种缓存存储可以使用，如memcached, redis，文件，数据库，…缓存管理器对Laravel说:好的，你现在将使用redis驱动程序，但如果用户说使用文件驱动程序，使用那个。有很多在驱动程序上工作的照亮组件:会话、身份验证、队列等。你可以在你的应用程序中使用相同的模式。让我给你举个例子，在我的脑海中:假设你正在对一些用户内容进行排名。您将创建一个名为Ranker的类，它扩展了“\Support\Manager”。你的应用程序中实现了多种排名算法，每一种算法实现都是所谓的驱动程序，你可以随意交换它们。它们都有相同的方法，由一些排序算法接口决定。有一天你想用一种算法，然后如果这个算法因为某种原因不能用了，就换另一个，等等。也许有更好的例子，但是好吧。有一个“照亮\支持\管理器”类允许这种基于驱动程序的实现。可以看到，如果要扩展这个类，需要实现' getDefaultDriver '方法来确定默认情况下将使用哪个驱动程序。在缓存的情况下，这是从配置文件中读取，以确定应用程序将使用哪个缓存存储。然后，一旦你在Ranker类上调用了一个方法，比如rankContent，如果默认的驱动程序是AlgorithmA, Ranker将从当前活动的驱动程序委托给rankContent方法。现在，当您想要更改另一个驱动程序时，请更改' getDefaultDriver '方法来使用AlgorithmB驱动程序，这样就完成了。Ranker类不关心正在使用哪个驱动程序，也不需要更改任何其他内容。*
>
> ### Registering drivers
>
> Before the manager calls methods on drivers, it needs to know about those drivers. We need to register them. When calling a method on Ranker class, it calls that method on whatever is returned from `driver()`. Now we noticed that `driver()` method should return an instance of current driver being used. The `driver()` method can be seen here and it's easy to see it's calling the createDriver method. The createDriver method performs logic that creates a new instance of the driver. It uses PHPs dynamic method name magic and requires this naming convention:
>
> *在管理器调用驱动程序的方法之前，它需要了解这些驱动程序。我们需要登记他们。当在Ranker类上调用一个方法时，它会在从' driver() '返回的任何东西上调用该方法。现在我们注意到' driver() '方法应该返回当前正在使用的驱动程序的实例。这里可以看到' driver() '方法，很容易看出它正在调用createDriver方法。createDriver方法执行创建驱动程序新实例的逻辑。它使用了PHPs动态方法的命名魔法，需要这样的命名约定:*
>
> ```php
> $method = 'create'.Str::studly($driver).'Driver';
> 
> if (method_exists($this, $method)) {
>  return $this->$method();
> }
> ```
>
> So, if you have drivers called AlgorithmA, AlgorithmB, etc. the methods that will be called and create them are: createAlgorithmADriver, etc. This method should return a new instance of the driver. Once it creates a new instance of drivers, the Ranker (Manager) will just call methods on that driver class.
>
> *所以，如果你有被称为AlgorithmA, AlgorithmB等的驱动程序，那么将会被调用并创建它们的方法是:createAlgorithmADriver，等等。这个方法应该返回一个驱动程序的新实例。一旦它创建了一个新的驱动程序实例，Ranker (Manager)就会调用这个驱动程序类的方法。*
>
> ### Extending with custom drivers
>
> Manager class provides you with extend method that works similarly as Macros, which allows a developer to register custom driver on the fly. Before a driver is created on the class, it's going to check if the custom driver exists and invoke it if it does.
>
> *Manager类为您提供了类似于宏的扩展方法，允许开发人员动态注册自定义驱动程序。在类上创建驱动程序之前，它将检查自定义驱动程序是否存在，如果存在，则调用它。*
>
> ## Arr & Str
>
> Laravel provides Arr and Str classes with a handful of methods to work on strings and arrays. Note that Str class is a big list of static methods, because strings are immutable, so it receives a string and returns a new modified version leaving the input one intact. Arr class is a helper class that performs operations on arrays, making it much easier for end user. Random element, last element, first element, nth element, cross join two arrays, convert to "dot" notation — all really easy with this class. Don't be scared to use them.
>
> *Laravel为Arr和Str类提供了一些处理字符串和数组的方法。注意，Str类是一个很大的静态方法列表，因为字符串是不可变的，所以它接收一个字符串并返回一个新的修改后的版本，而不改变输入。Arr类是一个助手类，它执行数组上的操作，这使得终端用户更容易使用它。随机元素，最后一个元素，第一个元素，第n个元素，交叉连接两个数组，转换成“点”符号-所有这些都很容易与这个类。不要害怕使用它们。*
>
> ## Collection
>
> I believe this requires no introduction. But note that there are 2 Collection classes. `Illuminate\Support\Collection` and `Illuminate\Database\Eloquent\Collection`. The latter is inherited from the former and returned when you retrieve items from the database. It contains some additional functionality, such as load (eager loading models' relationships), loadMissing (same thing but loading only non-loaded relationships), etc. It also holds the find method (note that this is not the same thing as `User::find($id)`). First one operates on a Collection instance (meaning all your data has already been retrieved from the database and second one executes the query that only selects the item with specific ID. By browsing through Eloquent Collection methods, you can find some cool stuff there, such as `modelKeys()` method that returns (plucks) only primary key (usually IDs) of the models, `fresh()` method that reloads the query. You'll find yourself using this mostly in your unit tests when a model changes but you need to get a fresh instance to see if the data has persisted in the database. There's also a lot of very useful methods such as `makeHidden` and `makeVisible` which hides certain attributes from the model for current instance of Collection (not for every instance). You can find more at the documentation.
>
> *我想这不需要介绍。但是请注意，有两个集合类。“照亮\支持\集合”和“照亮\雄辩的\ \数据库集合”。后者继承自前者，并在从数据库检索项时返回。它包含一些额外的功能，例如load(立即加载模型的关系)、loadMissing(相同的东西，但只加载未加载的关系)等等。它还保存了find方法(注意，这与' User::find($id) '不同)。第一个作用于一组实例(这意味着所有数据已经从数据库检索和第二个执行查询,只有选择项与特定的ID。通过浏览雄辩的收集方法,你可以找到一些很酷的东西,比如“modelKeys()方法,它返回(拔)只有主键(通常是IDs)的模型,新鲜的()的方法,重新加载查询。您会发现，当模型发生更改时，您将主要在单元测试中使用这个方法，但是您需要获取一个新的实例来查看数据是否已经持久化到数据库中。还有很多非常有用的方法，比如“makeHidden”和“makeVisible”，它们为当前集合实例(不是每个实例)的模型隐藏某些属性。您可以在文档中找到更多信息。*
>
> ## Custom collection
>
> If you didn't know, you can create a custom Collection class by extending the base one. This is useful if you need your models to be an instance of your own collection for some hacking. Jeffrey Way used an example of custom collections for threaded comments, there he would have a root method and nested method on his own collection. Then, if you need your models (e.g. User) to return instance of your own collection (e.g. UserCollection), override Laravel's `newCollection(array $models = [])` method in User model by simply returning `new UserCollection($models)`.
>
> *如果您不知道，您可以通过扩展基类来创建自定义集合类。如果您需要您的模型作为您自己的集合的实例来进行某些黑客活动，那么这是非常有用的。Jeffrey Way使用了一个自定义集合来处理线程注释的例子，他在自己的集合上有一个根方法和一个嵌套方法。然后，如果您需要您的模型(例如User)返回您自己的集合实例(例如UserCollection)，只需通过返回“new UserCollection($models =[]数组)”来覆盖用户模型中的`new UserCollection($models)`方法。*
>
> ## Helpers
>
> There's a ton of helper functions that can be very useful from time to time. And many you probably didn't know exist. For example: have you heard about `blank($value)` helper function? Or `value($value)`? Yeah, there's a lot of great helper functions, and as they are very well documented, I won't go too deep into it, but make sure to check them out.
>
> *有大量的辅助函数有时非常有用。很多你可能不知道。例如:你听说过“blank($value)”帮助函数吗?或“价值(美元值)”?是的，有很多很好的帮助函数，它们都有很好的文档，我不会深入介绍，但是一定要检查一下。*
>
> This will be it for today, I hope you learned something new about the framework and can find a good use for these classes and functions I showed you!
>
> 
>
> *今天就到这里，我希望您能学到一些关于框架的新知识，并能很好地使用我展示给您的这些类和函数!*

