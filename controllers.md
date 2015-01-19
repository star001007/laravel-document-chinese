# 控制器

### 基本的控制器
相比在路由中定义所有的逻辑，你可能更愿意把这些行为组织到控制器类里。控制器可以把相关的路由逻辑聚集到一个类里，并且可以充分发挥框架的特性，如：自动依赖注入。

控制器通常放在`app/controllers`文件夹下，这些文件默认在`composer.json`文件里的选项`classmap`
默认被注册了。当然你也可以放在其他位置，只需要保证`composer`可以自动载入这些类。

下面是一个简单的控制器类：
```php
class UserController extends BaseController {

    /**
     * Show the profile for the given user.
     */
    public function showProfile($id)
    {
        $user = User::find($id);

        return View::make('user.profile', array('user' => $user));
    }

}
```
所有的控制器类都必须继承`BaseController`类，这个类也在`app/controllers`里，它被用来存放一些公共的控制逻辑。`BaseControllers`类又继承了框架的`Controllers`类。
现在，你可以如下路由你的控制器方法：
```
Route::get('user/{id}', 'UserController@showProfile');
```
若你想用命名空间来组织和嵌套你的控制器，定义路由时需要使用完整的类命名：
```
Route::get('foo', 'Namespace\FooController@method');
```

>* 注意：因为我们是用`composer`来自动加载php类，控制器可以在任何地方，只要`composer`可以加载到。

你也可以给控制器路由指定名称：
```
Route::get('foo', array('uses' => 'FooController@method',
                                        'as' => 'name'));
```

通过`URL::action`方法或者action帮助函数可以获取控制器方法的url:
```php
$url = URL::action('FooController@method');

$url = action('FooController@method');
```

通过`Route::currentRouteAction`来获取当前控制器方法的名称：
```php
$action = Route::currentRouteAction();
```

### 控制器过滤器

过滤器同正匹配路由一样，指定到相对应的路由器：
```
Route::get('profile', array('before' => 'auth',
            'uses' => 'UserController@showProfile'));
```
当然，你也可以在控制器内部来指定过滤器：
```php
class UserController extends BaseController {

    /**
     * Instantiate a new UserController instance.
     */
    public function __construct()
    {
        $this->beforeFilter('auth', array('except' => 'getLogin'));

        $this->beforeFilter('csrf', array('on' => 'post'));

        $this->afterFilter('log', array('only' =>
                            array('fooAction', 'barAction')));
    }

}
```

你也可以通过匿名函数来指定过滤器：
```php
class UserController extends BaseController {

    /**
     * Instantiate a new UserController instance.
     */
    public function __construct()
    {
        $this->beforeFilter(function()
        {
            //
        });
    }

}
```
你也可以使用当前控制器的其他方法作为控制器，使用`@`前缀来定义：
```php
class UserController extends BaseController {

    /**
     * Instantiate a new UserController instance.
     */
    public function __construct()
    {
        $this->beforeFilter('@filterRequests');
    }

    /**
     * Filter the incoming requests.
     */
    public function filterRequests($route, $request)
    {
        //
    }

}
```

### 隐式控制器
laravel可以很容易的用一个路由器来处理一个控制器的所有方法，使用`Route::controller`如下：
```
Route::controller('users', 'UserController');
```
接收两个参数。第一个是控制器处理的base URI，第二个是控制器类名。接下里，我们给控制器每个方法添加`HTTP 方式`前缀：
```
class UserController extends BaseController {

    public function getIndex()
    {
        //
    }

    public function postProfile()
    {
        //
    }

    public function anyLogin()
    {
        //
    }

}
```
上例`index`方法对应的root URI 是`users`。

如果你的控制器方法有多个单词，你可以在url地址里通过`短斜线`来使用。比如URI为`users/admin-profile`对应控制器`UserController`的方法为：
```
public function getAdminProfile() {}
```

### RESTful 资源控制器

资源控制器可以很容易的创建关于`resources`的Restful控制器。比如，你可能需要创建一个控制器来处理应用中的`图片`。通过`Artisan CLI`和`Route::resource`可以构建这样一个控制器：
```
php artisan controller:make PhotoController
```
接下里，给这个控制器注册一个资源模式的路由：
```
Route::resource('photo', 'PhotoController');
```
这个路由定义创建了一系列对照片RESTful操作的路由。同样滴，生成的控制器也包含了一些列的方法，并有相对应的URI和请求方式。

资源控制器处理的方法：

|Verb|	Path|	Action|	Route Name|
| --------   | -----:  | :----:  |
|GET|	/resource|	index|	resource.index|
|GET|	/resource/create|	create|	resource.create|
|POST|	/resource|	store|	resource.store|
|GET|	/resource/{resource}|	show|	resource.show|
|GET|	/resource/{resource}/edit|	edit|	resource.edit|
|PUT/PATCH|	/resource/{resource}|	update|	resource.update|
|DELETE|	/resource/{resource}|	destroy|	resource.destroy|

如果你只需要其中一部分方法的话，可以这样:
```
php artisan controller:make PhotoController --only=index,show

php artisan controller:make PhotoController --except=index
```
也可以给路由指定一部分的方法：
```
Route::resource('photo', 'PhotoController',
                array('only' => array('index', 'show')));

Route::resource('photo', 'PhotoController',
                array('except' => array('create', 'store', 'update', 'destroy')));
```

默认地，资源控制器都有一个路由名称。你也可以通过一个下标为`names`的数组来覆盖默认名称：
```
Route::resource('photo', 'PhotoController',
                array('names' => array('create' => 'photo.build')));
```

#### 处理嵌套的资源控制器：
路由定义中，使用`.`符号来使用嵌套：
```
Route::resource('photos.comments', 'PhotoCommentController');
```
这个路由会注册一个嵌套的资源，你可以通过urls类似：`photos/{photoResource}/comments/{commentResource}`来访问。
```
class PhotoCommentController extends BaseController {

    public function show($photoId, $commentId)
    {
        //
    }

}
```
#### 给资源控制器添加其他路由
若需要给你的资源控制器添加其他路由，除了默认资源路由外，你需要在调用`Route::resource`之前来定义这些路由：
```
Route::get('photos/popular');
Route::resource('photos', 'PhotoController');
```

### 处理缺少方法
在使用`Route::controller`时，可以在控制器内部定义个捕获方,在没有匹配到控制器的方法时自动调用。这个方法名称为`misssingMethod`，并且接收请求的方法和参数数组。

定义如下：
```
public function missingMethod($parameters = array())
{
    //
}
```
在使用资源控制器时，你可能还需要在该控制器上定义`__call`魔术方法来调用`missingMethod`。
