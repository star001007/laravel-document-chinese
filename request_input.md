# 请求和输入

### 基本输入
只需要简单几个方法就可以获取用户所有输入。你不需要考虑用户HTTP请求方法，不管哪儿种请求方式，都是一样的获取方式。

#### 获取输入的值
```
$name = Input::get('name');
```

#### 如果输入值不存在，设定默认值
```
$name = Input::get('name', 'Sally');
```

#### 判断输入值是否存在
```
if (Input::has('name'))
{
    //
}
```

#### 获取请求的所有输入
```
$input = Input::all();
```

#### 获取请求的部分输入
```
$input = Input::only('username', 'password');   // 只有

$input = Input::except('credit_card');  //除了
```

若输入是数组时，你可以通过`.`符号来获取：
```
$input = Input::get('products.0.name');
```

>* 注意：一些Javascript库，如Backbone可能以JSON的形式发送，你也可以通过`Input::get`来访问。


### Cookies
laravel框架创建的cookie都是加密并且有认证码的签名，也就是客户端修改会造成cookie的无效。

#### 获取cookie值
```
$value = Cookie::get('name');
```

#### 给响应添加新的cookie
```
$response = Response::make('Hello World');

$response->withCookie(Cookie::make('name', 'value', $minutes));
```

#### 响应前cookie入队列
若想在响应创建前设置cookie，使用`Cookie::queue()`方法。这个`cookie`会在最终的响应中自动携带。
```
Cookie::queue($name, $value, $minutes);
```

#### 创建Cookie永久有效
```
$cookie = Cookie::forever('name', 'value');
```

### 原有输入
有时候，你可能想保存输入直到下次请求。比如，在检测验证错误后，需要重新填充表单。

#### 将输入刷入Session
```
Input::flash();
```
#### 将部分输入刷入Session
```
Input::flashOnly('username', 'email');

Input::flashExcept('password');
```

大多时候，可能你只需要将刷入跟跳转的前一个页面关联，那么你可以串联输入刷入到跳转上面。
```
return Redirect::to('form')->withInput();

return Redirect::to('form')->withInput(Input::except('password'));
```

#### 获取原有数据
```
Input::old('username');
```

### 文件

#### 获取一个上传文件
```
$file = Input::file('photo');
```

#### 判断文件是否上传
```
if (Input::hasFile('photo'))
{
    //
}
```

`Input::file`方法返回的对象是`Symfony\Component\HttpFoundation\File\UploadedFile`的实例，它继承了PHP的`SplFileInfo `方法，有文件交互的大量的方法。

#### 判断上传文件是否有效
```
if (Input::file('photo')->isValid())
{
    //
}
```

#### 移动上传文件
```
Input::file('photo')->move($destinationPath);

Input::file('photo')->move($destinationPath, $fileName);
```

#### 获取上传文件的路径
```
$path = Input::file('photo')->getRealPath();
```

#### 获取上传文件的原本名称
```
$name = Input::file('photo')->getClientOriginalName();
```

#### 获取上传文件的扩展名称
```
$extension = Input::file('photo')->getClientOriginalExtension();
```

#### 获取上传文件的大小
```
$size = Input::file('photo')->getSize();
```

#### 获取上传文件的MIME类型
```
$mime = Input::file('photo')->getMimeType();
```

### 请求信息
`Request`类提供许多方法来检查HTTP请求和扩展`Symfony\Component\HttpFoundation\Request`类，下面是其中一些：

#### 获取请求URI 
```
$uri = Request::path();
```

#### 获取请求方式
```
$method = Request::method();

if (Request::isMethod('post'))
{
    //
}
```

#### 判断请求是否匹配指定模式
```
if (Request::is('admin/*'))
{
    //
}
```

#### 获取请求URL
```
$url = Request::url();
```

#### 获取请求URI Segment
```
$segment = Request::segment(1);
```

#### 获取请求Header
```
$value = Request::header('Content-Type');
```

#### 获取$_SERVER数据
```
$value = Request::server('PATH_INFO');
```

#### 判断请求是否是HTTPS 
```
if (Request::secure())
{
    //
}
```

#### 判断请求是否是AJAX 
```
if (Request::ajax())
{
    //
}
```

#### 判断请求是否是JSON
```
if (Request::isJson())
{
    //
}
```


#### 判断请求是否请求JSON响应数据
```
if (Request::wantsJson())
{
    //
}
```

#### 获取请求响应的数据格式
`Request::format`方法基于HTTP Accept请求头来返回响应格式：
```
if (Request::format() == 'json')
{
    //
}
```
