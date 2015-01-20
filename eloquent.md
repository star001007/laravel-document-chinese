# Eloquent ORM

### 介绍

Laravel包含的Eloquent ORM提供了处理数据库的一套优美、简易的ActiveRecord实现。每个数据表有一个与之对应的`模型`来操作数据表。

开始之前，确认你配置好了`app/config/database.php`中的数据库连接。

### 基本用法
首先，让我们创建一个Eloquent模型，默认在`app/models`文件夹下，你也可以在`composer.json`文件里随意指定它的位置，只要确保被自动加载。

#### 定义Eloquent模型
```
class User extends Eloquent{}
```

注意，我们并没有特意指定这个模型使用的数据表。其实这个类名的小写和复数就是默认的数据表，除非你特意指定。上面`User`模型对应的数据表为`users`，若需要自定义表名，在这个模型类中添加`protected`的属性`$table`即可：
```php
class User extends Eloquent{
    protected $table = 'my_users';
}
```
>* 注意：Eloquent默认假设每个表有一个名为`id`的主键，你可以定义`primaryKey`属性来覆盖它。同样地，使用模型时用到的数据库连接，你也可以定义`connection`属性来覆盖它。

一旦模型被定义，你就可以获取和创建数据表记录。注意，默认你需要给数据表指定`updated_at`和`created_at`列，如果你希望自动更新，设置`$timestamps`属性为`false`;

#### 获取所有数据
```php
$users = User::all();
```

#### 通过主键来获取一条记录
```php
$user = User::find(1);

var_dump($user->name);
```
>* 注意：所有查询生成器的方法，Eloqunet模型也可以使用。

#### 通过主键获取一条数据，失败抛出异常
有时候你希望模型未找到时抛出一个异常，这样就可以用`App:error`处理器来捕获它，并展示404页面：
```php
$model = User::findOrFail(1);

$model = User::where('votes', '>', 100)->firstOrFail();
```
注册错误处理器，监听` ModelNotFoundException`:
```php
use Illuminate\Database\Eloquent\ModelNotFoundException;

App::error(function(ModelNotFoundException $e)
{
    return Response::make('Not Found', 404);
});
```

#### query使用Eloquent模型
```
$users = User::where('votes', '>', 100)->take(10)->get();

foreach ($users as $user)
{
    var_dump($user->name);
}
```

#### Eloquent聚合
你也可以使用查询生成器的聚合方法：
```
$count = User::where('votes', '>', 100)->count();
```
如果你不能通过查询生成器来生成query，你可以使用`whereRaw`:
```
$users = User::whereRaw('age > ? and votes = 100', array(25))->get();
```

#### 拆分查询
若需要处理大量（数以千计）的Eloquent记录，使用`chunk`方法可以防止吃掉太多的内存：
```
User::chunk(200, function($users)
{
    foreach ($users as $user)
        {
            //
        }
});
```
第一个参数200即时你每个`chunk`处理的记录数。闭包回调参数`$users`是数据库查询的记录，查询完成后自动调用。


#### 指定某个query的数据库链接
使用`on`方法可以指定某个数据库连接以查询这个query
```
$user = User::on('connection-name')->find(1);
```