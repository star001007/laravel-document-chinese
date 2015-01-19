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























