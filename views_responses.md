# 视图和响应

### 基本响应 Response

#### 路由返回字符串
```
Route::get('/', function()
{
    return 'Hello World';
});
```

#### 创建自定义响应

`Response`的实例继承自`Symfony\Component\HttpFoundation\Response`类，提供了很多方法用来构建HTTP响应。
```
$response = Response::make($contents, $statusCode);

$response->header('Content-Type', $value);

return $response;
```

如果你想用`Response`类方法，但是返回一个视图作为响应。那么，你可以使用`Response::view`方法：
```
return Response::view('hello')->header('Content-Type', $type);
```

#### 响应携带Cookies
```
$cookie = Cookie::make('name', 'value');

return Response::make($content)->withCookie($cookie);
```


### 跳转 Redirects

#### 返回一个跳转
```
return Redirect::to('user/login');
```

#### 返回一个带有（Flash Data）的跳转
```
return Redirect::to('user/login')->with('message', 'Login Failed');
```
>* 注意：`with`方法会讲数据闪存到session里，你可以通过Session::get方法来获取。

#### 返回一个命名路由的跳转
```
return Redirect::route('login');
```

#### 返回一个携带参数的命名路由的跳转
```
return Redirect::route('profile', array(1));
```

#### 返回一个携带命名参数的命名路由的跳转
```
return Redirect::route('profile', array('user' => 1));
```

#### 返回一个控制器方法的跳转
```
return Redirect::action('HomeController@index');
```

#### 返回一个控制器方法携带参数的跳转
```
return Redirect::action('UserController@profile', array(1));
```

#### 返回一个控制器方法携带命名参数的跳转
```
return Redirect::action('UserController@profile', array('user' => 1));
```

### 视图 Views
视图包含应用的html代码，使得展现与控制和逻辑分离。视图都放在`app/views`文件夹下。

下面展示一个简答的视图：
```
<!-- View stored in app/views/greeting.php -->

<html>
    <body>
        <h1>Hello, <?php echo $name; ?></h1>
    </body>
</html>
```
这个视图可以如下发送到浏览器端：
```
Route::get('/', function()
{
    return View::make('greeting', array('name' => 'Taylor'));
});
```

#### 传递数据到视图
```
// Using conventional approach
$view = View::make('greeting')->with('name', 'Steve');

// Using Magic Methods
$view = View::make('greeting')->withName('steve');
```
上面代码里的`$name`变量将在视图`greeting.php`中可用，其值为`steve`。你也可以传递一个数组作为`make`方法的第二个参数：
```
$view = View::make('greetings', $data);
```

你还可以共享数据给所有的视图：
```
View::share('name', 'Steve');
```

#### 传递一个子视图到一个视图
有时候需要传递一个视图到另一个视图。比如，一个子视图存在在`app/views/child/view.php`，那么我们可以像下面传递：
```
$view = View::make('greeting')->nest('child', 'child.view');

$view = View::make('greeting')->nest('child', 'child.view', $data);

```
子视图可以在父视图中被渲染：
```
<html>
    <body>
        <h1>Hello!</h1>
        <?php echo $child; ?>
    </body>
</html>

```

### 视图组件
视图组件就是在视图渲染时调用回调函数或者类方法。 如果你想在每次渲染视图时，绑定数据到视图，你可以组织这些代码到一个地方。形象地说，视图组件更像是`视图模型` 或者 `表现者`。

#### 定义一个视图组件
```
View::composer('profile', function($view)
{
    $view->with('count', User::count());
});
```

现在，每次`profile`视图被渲染时，`count`数据都会被绑定到视图中。

你也可以绑定一个视图组件到多个视图里：
```
View::composer(array('profile','dashboard'), function($view)
{
    $view->with('count', User::count());
});
```
若你想使用类组件而不是匿名函数，也有助于使用`Ioc`容器来解析，使用如下：
```
View::composer('profile', 'ProfileComposer');
```

一个类组件可以像下面这样定义：
```
class ProfileComposer {

    public function compose($view)
    {
        $view->with('count', User::count());
    }

}
```

#### 定义多个视图组件 
你可以使用`composer`方法来一次注册多个视图组件：
```
View::composers(array(
    'AdminComposer' => array('admin.index', 'admin.profile'),
    'UserComposer' => 'user',
    'ProductComposer@create' => 'product' 
));
```
>* 注意：类组件没有约束放到哪儿里，你可以随意指定，只要它能被composer.json文件里自动加载使用。 


#### 视图创建者
视图创建者类似视图组件，唯一区别就是它会在视图实例化时就会激活。使用`creator`方法：
```
View::creator('profile', function($view)
{
    $view->with('count', User::count());
});
```

### 特殊响应

### 创建一个json响应
```
return Response::json(array('name' => 'Steve', 'state' => 'CA'));
```

### 创建一个jsonp响应
```
return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));
```

### 创建一个文件下载响应
```
return Response::download($pathToFile);

return Response::download($pathToFile, $name, $headers);
```
>* 注意：Symfony HttpFoundation处理文件下载时，限定下载文件名必须是ASCII编码的。

### 响应Macros

若你想自定义一个响应可以在路由和控制器中多次使用，你可以使用`Response::macro`方法：
```
Response::macro('caps', function($value)
{
    return Response::make(strtoupper($value));
});
```
第一个参数作为自定义响应的名称，第二个匿名函数会在`Response`类调用这个响应时被调用:
```
return Response::caps('foo');

```
你可以在`app/start`文件里选一个定义，又或者你也可以新建一个文件来组件这些`macros`，这个文件被`app/start`文件里的其中任何一个所引入。

