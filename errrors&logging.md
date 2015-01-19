# 错误和日志

### 配置
应用的日志处理程序在启动文件`/app/start/global.php`里注册。默认，它是单个日志文件，如果你想根据需要配置，你可以使用laravel有名的`Monolog`日志库文件，可以充分利用这个库提供的方法。

比如，如果你想每天生成一个文件，那么你可以如下配置：
```php
$logFile = 'laravel.log';

Log::useDailyFiles(storage_path().'/logs/'.$logFile);
```

#### 错误详细信息
默认，错误详细信息是开启的。当有错误发生时，你将会看到错误的调用栈过程和错误信息。你也可以设置`app/config/app.php`里选项`debug`的值为`false`来禁用它。
>* 注意：生成环境墙裂建议您关闭错误信息调试。

### 错误信息处理
默认地，`app/start/global.php`文件包含了针对所有异常的错误处理器：
```php
App::error(function(Exception $exception)
{
    Log::error($exception);
});
```

这是最最基本的错误处理器。你也可以根据需要来定义，处理器一般是根据异常指定的类型提示来调用。比如，我们创建一个处理器只接受`RuntimeException`异常的实例：
```php
App::error(function(RuntimeException $exception)
{
    // Handle the exception...
});
```
若异常处理器返回一个响应，这个响应会被发送到浏览器其他从错误异常将不会被调用：
```php
App::error(function(InvalidUserException $exception)
{
    Log::error($exception);

    return 'Sorry! Something is wrong with this account!';
});
```
若想监听PHP`fatal errors`，你需要使用`App::fatal`方法：
```php
App::fatal(function($exception)
{
    //
});
```
若有多个异常处理器，则异常处理器应该定义应该是从一般到具体。比如，`Exception`异常应该定义在自定义异常`Illuminate\Encryption\DecryptException`：

#### 异常处理器放到哪儿
laravel没有指定默认地宿主目录，你可以随意指定。一个选择时放在`start/global.php`文件。总体来说，这里是存放启动代码的绝佳选择。若这个文件越来越臃肿，你可以创建`app/errors.php`文件，然后在`start/global.php`文件里导入它。第三种方法是创建一个`服务器提供商`来注册和处理。没有唯一标准，选择你习惯的就好。



### HTTP异常
一些异常描述了服务器的HTTP错误代码。比如，"page not fountd"404错误，"unauthorized error"401错误，或者500内部错误。若需要产生这些响应时，使用如下方法：
```
App:abort(404);
```

同时地，你也可以提供一个响应
```
App::abort(403, 'Unauthorized action.');
```
这个方法可以在请求生命周期内任何时间内使用。


### 处理404错误
若需要方便地返回自定义的404错误页面，你可以给应用中所有"404 Not Found"错误注册一个错误处理器：
```
App::missing(function($exception)
{
    return Response::view('errors.missing', array(), 404);
});
```

### 日志
laravel日志工具在强大的`Monolog`库文件上面提供了一个简单的层。默认地，laravel是配置成单个日志文件的，存储在`app/storage/logs/laravel.log`。你可以如下给日志系统里写入日志：
```php
Log::info('This is some useful information.');

Log::warning('Something could be going wrong.');

Log::error('Something is really going wrong.');
```
这个日志工具根据`RFC 5424`定义：**debug**,**info**,**notice**,**warning**,**error**,**critical**和**alert** 提供了6个日志级别：

日志方法可以传递一个`context`索引的数组：
```
Log::info('Log message', array('context' => 'Other helpful information'));
```

Monolog还有一系列的额外处理器你可以用来记录日志。若想获取部署的Monolog实例，下面调用：
```
$monolog = Log::getMonolog();
```
你也可以给传递给日志的所有信息注册一个事件：

#### 注册日志监听
```
Log::listen(function($level, $message, $context)
{
    //
});
```




























