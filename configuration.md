# 配置

### 介绍
Laravel框架的所有配置信息都在`app/config`目录里。每个选项都有详细的文档说明。你可以先熟悉一下。

有时你可能想在程序执行时获取这些配置的值，使用`Config`类。

#### 获取某个配置的值
```php
Config::get('app.timezone');
```
如果配置选项不存在时候，你可以指定一个默认值，如下：
```php
$timezone = Config::get('app.timezone', 'UTC');
```

#### 设置某个配置的值
可能你已经注意到了，`.`分隔符用来获取多个文件中的值。
```php
Config::set('database.default', 'sqlite');
```
不过，它只在当前请求中有效。

### 环境配置
对于不同的运行环境，一般会有不同的配置选项。比如，线上线下环境你可能使用不同的缓存驱动器。通过环境配置选项可以很容易的来实现。

只需要在`config`目录里创建一个新的文件夹，命名为与环境相关的名称。接下来，创建你想覆盖和配置一些选项的配置文件。比如你想覆盖线下环境里的缓存驱动器，你可以在 `app/config/local`里创建`cache.php`文件，内容如下：
```php
<?php

    return array(
        'driver' => 'file',
    );
```
```
注意：不要使用 testing 作为环境的名称。因为它是单元测试的名称。
```

你会发现你并没有指定配置文件里所有的选项，而只是你想覆盖的一些选项。因为它会继承这些基本的配置文件。

接下里，你要指定框架使用哪儿个环境。默认地是`production`，即生产环境。不过，你也可以在`bootstrap/start.php`文件里指定使用环境。这个文件里，你会发现`$app->detectEnvironment`调用，这个数组会决定你使用哪儿个环境。你也可以根据需要给这个数组添加其他环境和机器名称。
```php
<?php
$env = $app->detectEnvironment(array(
    'local' =>  array('your-machine-name'),
));
```

上面例子中，`local`是你的环境的名称，`your-machine-name`是你的服务器的主机名。在linux或者Mac系统树上，你也可以通过终端命令：`hostname`来指定你的主机名。

如果你想要更灵活的检测环境。你可以给`detectEnvironment`方法传递一个闭包，可以根据你的意愿来是指定环境。
```php
$env = $app->detectEnviroment(function(){
    return $_SERVER['MY_LARAVEL_ENV'];
});

```

#### 获取当前应用的环境
使用`environment`方法如下：
```
$environment = App::environment();
```
也可以给`environment`方法传递一个环境参数来确定是否是当前环境。
```
if (App::environment('local'))
{
    // The environment is local
}

if (App::environment('local', 'staging'))
{
    // The environment is either local OR staging...
} 

```

#### Provider 配置
在使用环境配置时候，若想给在基础的`app`配置文件中追加环境`service providers`时候，直接追加会覆盖基础的 `app providers`。若想强制的追加，你需要在环境的app配置文件里使用`app_config`辅助方法如下：
```
'providers' => append_config(array(
    'LocalOnlyServiceProvider',
))
```

### 敏感数据保护的配置
对于生产环境来说，我们强烈建议您将所有的敏感信息的配置都放到配置文件之外。比如数据库密码，api密钥，及编码密钥应该尽可能的移到配置文件之外。 那么，我们应该放到哪儿里呢？ 幸运地是，laravel给我们提供了一个简单的解决办法，就是使用"点"文件(dot files)。

首先，配置你的应用来识别你的机器作为本地环境的机器。接下来，在你项目的根目录创建`.env.local.php`文件，就在`composer.json`文件同一目录里。这个文件返回一个键值对的数组，如所有的laravel配置文件一样：
```php
<?php

    return array(
        'TEST_STRIPE_KEY' => 'super-secret-sauce',
    );

```
这个文件的所有键值对都可以通过$_ENV 和$_SERVER 超全局变量访问。现在你可以在你的配置文件里访问这些超全局变量:
```php

'key' => $_ENV['TEST_STRIPE_KEY']

```
记住将`.env.local.php`文件添加到`。gitignore`文件里，以方便团队开发时，使用各自的配置文件同时保护自己的敏感数据。

现在，你也可以在你的生产环境的项目根目录中创建`.env.php`文件，包含对应的值。同样滴，在版本控制器中忽略它。

```
* 注意：你可以为每个环境创建一个配置文件。在开发环境创建`.env.development.php`,在生成环境中创建`.env.php`文件。
```


### 维护模式

当你的项目是在维护模式下，一个自定义的视图可以在应用中所有的路由里使用。当你更新或者进行维护时，可以很方便的禁用您的应用程序。在`app/start/global.php`文件中已经提供了`App::down`的方法，它的响应会在你启用维护模式后发给所有的用户。

启用维护模式，执行下列命令：
```
php artisan down
```
禁用维护模式，执行下列命令：
```
php artisan up
```
自定义视图的使用，需要在`app/start/global.php`里设置下：
```
App::down(function()
{
    return Response::view('maintenance', array(), 503);
});
```
如果down的匿名函数返回`null`,那么维护模式将会被所有请求所忽略。

#### 维护模式&队列
如果你的应用在维护模式中，那么队列任务将不会被执行。一旦维护模式结束后，队列任务将开始正常执行。





















