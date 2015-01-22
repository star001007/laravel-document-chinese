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

### 批量赋值（Mass Assignment）
创建模型后，我们可以传递一个属性数组给模型构造器。这些属性数组通过`Mass Assignment`赋值给这个模型。虽然非常方便，但是，盲目的赋值可能会有安全的隐患。如果用户可以随意的传递输入，就可以随意更改模型的属性。出于这个考虑，所有的模型会防范批量赋值。

开始前，需要给模型设置`fillable` 和 `guarded`属性：

#### 给模型定义Fillable属性
`fillable`属性指定了批量赋值哪儿些属性。可以在类或者实例类设置：
```php
class User extends Eloquent {
    protected $fillable = array('first_name', 'last_name', 'email');
}
```
如上，只有列出的三个属性可以进行批量赋值。

#### 给模型定义Guarded属性
与`fillable`相反的是`guarded`，类似于黑名单一样。
```php
class User extends Eloquent{
    protected $guarded = array('id', 'password');
}
```
>* 注意：当使用`guarded`时，你不可以使用`Input::get()`或者任何其他用户输入的原生数组给`save`和`update`方法，因为没在`guarded`中的字段可能被更新。

#### 阻止批量赋值里的所有属性
上例中，`id`和`password`属性不能批量赋值，而其他的可以。若想阻止所有属性：
```php
protected $guarded = array('*');
```

### 插入、更新、删除

#### 添加一条记录
```
$user = new User;

$user->name = 'John';

$user->save();
```
>* 注意：通常情况下，Eloquent模型有自增键。如果你需要指定自己的，在模型中设置`incrementing`属性为`false`。

你可以使用`create`方法，使用一行代码就可以创建一个新的模型。`create`方法会返回插入模型的实例。当然了，你必须指定属性`fillable`或者`guarded`，因为所有的`Eloquent`模型都防范批量赋值。

使用自增id保存或者新建模型实例后，你可以通过`id`属性来获取它的值：
```php
$insertedId = $user->id;
```

#### 给模型设置`Guarded`属性
```php
class User extends Eloquent{
    protected $guarded = array('id', 'account_id');
}
```

#### 使用模型的`create`方法
```php
// Create a new user in the database...
$user = User::create(array('name' => 'John'));

// Retrieve the user by the attributes, or create it if it doesn't exist...
$user = User::firstOrCreate(array('name' => 'John'));

// Retrieve the user by the attributes, or instantiate a new instance...
$user = User::firstOrNew(array('name' => 'John'));
```


#### 获取模型后更新
你可以先获取一个模型实例，改变它的属性值，然后用`save`方法来更新：
```php
$user = User::find(1);

$user->email = 'john@foo.com';

$user->save();
```

#### 保存模型及关系
若需要保存一个模型实例及所有关系，可以使用`push`方法：
```php
$user->push();
```

你也可以以query的方式来执行udpate
```php
$affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));
```
>* 注意：当你使用Eloquent的查询生成器时，不会激活任何的事件

#### 删除存在的模型实例
调用`delete`方法：
```php
$user = User::find(1);

$user->delete();
```

#### 通过主键值来删除模型实例
```php
User::destroy(1);

User::destroy(array(1, 2, 3));

User::destroy(1, 2, 3);
```

当然你可以在一系列的模型实例上执行一个删除query：
```
$affectedRows = User::where('votes', '>', 100)->delete();
```

#### 只更新模型的timestamps字段
若纸箱更新模型的`timestamps`字段，使用`touch`方法：
```php
$user->touch();
```

### 软删除
当执行软删除时，并不会真正的从数据库删除数据，而是在记录的`deleted_at`时间戳进行设置。使用它需要在模型里应用`SoftDeletingTrait`：
```
user Illuminate\Database\Eloquent\SoftDeletingTrait

class User extends Eloquent{

    use SoftDeletingTrait;

    protected $dates = ['deleted_at'];
}
```
数据表添加`deleted_at`字段后，就可以在数据库迁移时用`softDeletes`方法：

```php
$table->softDeletes();
```
现在在模型上使用`delete`方法时，对应字段`deleted_at`值会设置为当前时间戳。当查询一个使用了软删除的模型时，“软删除”的数据不会包含到结果中。

#### 强制软删除数据显示到结果集中
若想强制显示软删除数据，query时使用`withTrashed`方法：
```php
$user = User::withTranshed()->where('account_id', 1)->get();
```

`withTranshed`方法在关联时依然有效：
```
$user->posts()->withTrashed()->get();
```
















