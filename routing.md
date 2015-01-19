# 路由

### 基本路由
应用大部分的路由都在`app/routes.php`文件里定义。最简单的路由有一个url和必包回调组成。

#### 基本GET路由
```php
Route::get('/', function()
{
    return 'Hello World';
});
```

#### 基本的POST路由
```php
Route::post('foo/bar', function()
{
    return 'Hello World';
});
```

#### 注册一个路由到多种HTTP方式
```
Route::match(array('GET', 'POST'), '/', function()
{
    return 'Hello World';
});
```
#### 注册一个路由到所有的HTTP方法
```
Route::any('foo', function()
{
    return 'Hello World';
});
```
#### 强制路由必须是HTTPS方式
```
Route::get('foo', array('https', function()
{
    return 'Must be over HTTPS';
}));
```
如果你想获取路由生成的url，可以通过`URL::to`方法来获取：
```
$url = URL::to('foo');

```

### 路由参数
```
Route::get('user/{id}', function($id)
{
    return 'User '.$id;
});
```
#### 可选路由参数
```
Route::get('user/{name?}', function($name = null)
{
    return $name;
});
```
#### 可选路由参数，有默认值
```
Route::get('user/{name?}', function($name = 'John')
{
    return $name;
});
```
#### 正则表达式，路由约束（通过where）
```
Route::get('user/{name}', function($name)
{
    //
})
->where('name', '[A-Za-z]+');

Route::get('user/{id}', function($id)
{
    //
})
->where('id', '[0-9]+');
```
#### where数组，路由约束
```
Route::get('user/{id}/{name}', function($id, $name)
{
    //
})
->where(array('id' => '[0-9]+', 'name' => '[a-z]+'))
```

#### 定义全局模式

如果你想对某一个路由参数始终用某一个特定表达式来约束，那么你可以使用`pattern`方法：
```
Route::pattern('id', '[0-9]+');

Route::get('user/{id}', function($id)
{
    // Only called if {id} is numeric.
});
```
#### 获取路由参数的值
如果想在路由外面使用路由参数，你可以通过`Route::input`方法来获取：
```
Route::filter('foo', function()
{
    if (Route::input('id') == 1)
    {
        //
    }
});
```

### 路由过滤器

路由过滤器可以通过限制路由很容易地帮你实现网站的验证。laravel框架有`auth`过滤器，`auth.basic`过滤器，`gust`过滤器和`csrf`过滤器，它们都在`app/filters`文件夹下。

#### 定义一个路由过滤器
```
Route::filter('old', function()
{
    if (Input::get('age') < 200)
    {
        return Redirect::to('home');
    }
});
```
如果这个过滤器返回一个响应，那么会作为请求的响应，这个路由器将不会被执行。这个路由的`after`过滤器也会取消执行。

#### 给路由绑定过滤器
```
Route::get('user', array('before' => 'old', function()
{
    return 'You are over 200 years old!';
}));
```
#### 给控制器绑定过滤器
```
Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));

```
#### 一个路由绑定多个过滤器
```
Route::get('user', array('before' => 'auth|old', function()
{
    return 'You are authenticated and over 200 years old!';
}));
```

#### 一个路由绑定多个过滤器（通过array）
```
Route::get('user', array('before' => array('auth', 'old'), function()
{
    return 'You are authenticated and over 200 years old!';
}));
```
#### 指定路由参数
```
Route::filter('age', function($route, $request, $value)
{
    //
});

Route::get('user', array('before' => 'age:200', function()
{
    return 'Hello World';
}));
```
`after`过滤器会接收`$response`作为郭略器的第三个参数：
```
Route::filter('log', function($route, $request, $response)
{
    //
});
```
#### 基于模式的过滤器（when方法）
你可以根据url，对某一系列的路由指定一个过滤器：
```
Route::filter('admin', function()
{
    //
});

Route::when('admin/*', 'admin');
```
上面例子中，对于所有以`admin/`开头的路由都使用`admin`过滤器。`*`符号会作为通配符，会匹配所有组合的字符。

你还可以通过HTTP方式来约束：
```
Route::when('admin/*', 'admin', array('post'));
```
#### 过滤器类
更高级的过滤器是用一个类替代闭包函数。过滤器类由应用的`Ioc`容器来解析，那么你可以在过滤中利用依赖注入，极大方便测试。

#### 注册一个类过滤器
```
Route::filter('foo', 'FooFilter');
```
默认地，会调用`FooFilter`类的`filter`方法：
```
class FooFilter {

    public function filter()
    {
        // Filter logic...
    }

}
```
如果你想调用其他方法，指定它就行：
```
Route::filter('foo', 'FooFilter@foo');

```

### 路由命名

命名路由在路由生成跳转和urls时候，指向路由更方便。你可以如下给路由命名：
```
Route::get('user/profile', array('as' => 'profile', function()
{
    //
}));
```
你可以给控制器的操作来命名：
```
Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));

```
那么你就可以使用如下：
```
$url = URL::route('profile');

$redirect = Redirect::route('profile');
```
你可以通过`currentRouteName `方法来获取当前路由的名称
```
$name = Route::currentRouteName();

```

### 路由组
当你需要给一组路由指定过滤器时，而不是每个路由都去指定时候，可以使用路由组：
```
Route::group(array('before' => 'auth'), function()
{
    Route::get('/', function()
    {
        // Has Auth Filter
    });

    Route::get('user/profile', function()
    {
        // Has Auth Filter
    });
});
```

你也可以在路由组的数组里使用`namespace`参数来限定这个组的控制器，保证它们在同一个命名空间内。
```
Route::group(array('namespace' => 'Admin'), function()
{
    //
});
```

### 子域名路由
laravel路由也可以处理通配符子域名，可以传递通配符参数。

#### 注册子域名路由
```
Route::group(array('domain' => '{account}.myapp.com'), function()
{

    Route::get('user/{id}', function($account, $id)
    {
        //
    });

});
```
### 路由前缀

组路由可以通过组array的属性选项`prefix`使用前缀:
```
Route::group(array('prefix' => 'admin'), function()
{

    Route::get('user', function()
    {
        //
    });

});
```
### 路由模型绑定
模型绑定可以方便的将模型实例注入到路由中。比如，你可以注入一个用户ID对应的模型实例，而不单单是一个用户ID 。首先，`Route::model`方法指定一个给定参数的模型。

#### 绑定一个参数到模型
```
Route::model('user', 'User');
```
接下来，定义一个含有`{user}`参数的路由如下：
```
Route::get('profile/{user}', function(User $user)
{
    //
});
```
因为我们之前已经将`{user}`参数绑定到了`User`模型，所以`User`实例将会注册到这个路由。也就是说，一个请求`profile/1`将会注册一个ID是1的用户实例。

>* 注意：如果匹配的模型实例在数据库中没有找到，那么会抛出一个404错误。

如果你想指定`not found`行为，你可以给`model`方法的第三个参数的传递一个
闭包函数：
```
Route::model('user', 'User', function()
{
    throw new NotFoundHttpException;
});
```

如果你想自己实现路由参数的解析，你可以使用`Route::bind`方法：
```
Route::bind('user', function($value, $route)
{
    return User::where('name', $value)->first();
});
```

### 抛出404错误

路由里，有两种方法可以手动触发404错误。第一种是：
```
App::abort(404);
```
第二种是抛出异常：`Symfony\Component\HttpKernel\Exception\NotFoundHttpException`的实例。

更多关于404异常信息及自定义错误响应在[errors](http://laravel.com/docs/errors#handling-404-errors)里说明。

### 路由到控制器
laravel不仅可以路由到闭包，也可以路由到控制器类，甚至到创建的`resource`控制器。

更多信息，看[控制器](http://laravel.com/docs/4.2/controllers)。







