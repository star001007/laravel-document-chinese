# 请求的生命周期

### 概述
所谓工欲善其事必先利其器。网站应用开发也是如此，理解工具的机理对你很有帮助，你会更有自信和更随心的使用它们。本文档的目的就是让你全面的、深刻的理解laravel框架的工作机理。随着你的理解加深，揭开它的迷纱，就可以更自如的创建应用。我们从 "启动文件"和 应用事件 来开始我们的了解。

刚开始你可能不是理解所有的东西，没关系，不要灰心。你只需要领会基本的东西，随着你阅读其他章节你会慢慢的理解它。

### 请求的生命周期
所有的请求都是从`public/index.php`入口的。当我们使用Apache时，`.htaccess`文件会帮助我们讲所有的请求指向index.php文件。从这里开始，laravel开始处理请求并作出相应。为此，我们从laravel的启动过程开始。

到目前为止，我们了解到laravel启动过程的最重要的是服务提供商（service providers）。在`/app/config/app.php`配置文件里，提供了一系列的服务提供商，还有一个providers数组。这些提供商作为larvel启动机制的主要部分。再回到`index.php`文件，一个请求进入`index.php`文件后，会加载`bootstrap/start.php`文件，这个文件创建了laravel应用对象，同时是Ioc容器。

创建应用对象后，需要设置几个项目路径和环境检测。接着颞部的启动脚本会被执行，这个文件作用于laravel内部代码，根据你的配置文件的设置来进行设置，比如`timezone`、`error_reporting` 等。更重要的，不是这些选项的设置，而是对应用中配置的服务器提供商(`service providers`)进行注册。

简单的服务器提供商只有一个方法：`register`。应用对象通过自己的`register`方法注册`service providers`时候，`service providers`的`register`方法才会被调用。一般来讲，每个`service provider`绑定一个或者多个闭包到容器里。在容器里允许你访问这些绑定的服务商。比如，`QueueServiceProvider`服务器服务注册了一些闭包，来解决涉及类的一些队列。当然，服务器提供商也可以再启动任务里使用而不仅仅是注册到容器里。服务器提供商可以注册 为事件监听、视图组件、`Artisan`命令等。

当所有的服务提供商被注册后，`app/start`文件会被加载。最后，`app/routes.php`文件会被调用。一旦`routes.php`文件被加载后，请求对象会被转发给应用以方便它转发给其他的路由。


让我们总结一下：
>1. 请求进入`public/index.php`文件
>1. `bootstrap/start.php`文件创建一个应用对象并进行环境监测
>1. 内部`framework/start.php` 文件配置选项并加载服务器提供商
>1. 应用`app/start`文件会被加载
>1. 应用`app/routes.php`文件会被加载
>1. 请求对象转发给应用，并返回响应对象
>1. 响应对象发送到终端

现在，你对于一个进入laravel的请求如何被处理有个大致的了解。接下来，我们看下启动文件。


### 启动文件

你的应用的启动在`app/start.php`里。默认地，会包含三个文件：`global.php`,`local.php`,`artisan.php`。对于`artisan.php`更多细节，查看[Artisan Comman Line](http://laravel.com/docs/4.2/commands#registering-commands)。

`global.php`文件默认包含了一些基本选项，比如：`Logger`的注册、引入`app/filters.php`文件。你也可以根据需要自己添加，它会在每个请求中都引入。`local.php`文件只有在`local`环境里被调用。关于环境更多信息，请看[配置文档](http://laravel.com/docs/4.2/configuration)。

当然，如果你有其他的环境，你也可以创建响应的启动文件，它会在对应环境中自动引入。比如，你配置文件`bootstrap/start.php`设置为`development`环境，那么这个应用当前环境下的所有请求都会引入`app/start/development.php`文件。

#### 启动文件里放什么内容
启动文件就是用来存放启动程序的相关代码，比如，你可以注册一个视图组件、配置你的日志选项及一些PHP的设置等。所有的一切，你都可以自由设置。但是，对于大型的项目来说，太多的启动代码会使得启动文件都别混乱，这时候你可以考虑将一些代码移到 服务器提供商。

### 应用事件

#### 注册应用事件
捏可以在请求的前后的过程中，注册`before`、`after`、`finish`和`shutdown`应用事件：

```php

App::before(function($request)
{
    //
});

App::after(function($request, $response)
{
    //
});

```

这些事件的监听会在应用每个请求的`before` 和`after` 被调用，它可以用来过滤或者修改请求的响应。你可以注册他们到启动文件中或者一个服务器提供商里。

你也可以注册监听到`matched`事件上，当一个请求匹配到一个未执行的路由上，则会被调用：
```php
Route::matched(function($route, $request)
{
    //
});

```

`finish`事件会在响应被发送到终端后才被调用，它可以用来处理应用最后的任务。`shutdown`事件则在所有的`finish`事件处理完所有事件后会被调用，它是脚步执行终止之前最后的处理机会。大多时候，你不需要用到这些事件。
