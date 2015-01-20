# 查询生成器

### 介绍
查询生成器提供了方便流畅的接口来创建和执行queries。它可以执行你应用的大多数的数据库操作，并且适配于所有支持的数据库。

>* 注意：Laravel的查询生气使用PDO的参数绑定来保护您的应用防止sql注入，绑定的参数不需要额外进行过滤处理。

### Selects

#### 获取表的所有记录
```
$users = DB::table('users')->get();

foreach ($users as $user)
{
    var_dump($user->name);
}

```

#### 获取表的一条记录
```
$user = DB::table('users')->where('name', 'John')->first();

var_dump($user->name);
```

#### 获取表的某行的一列数据
```
$name = DB::table('users')->where('name', 'John')->pluck('name');
```

#### 获取表的某列数据的列表
```
$roles = DB::table('roles')->lists('title');
```
这个方法会返回`role title`的数组。你可以给返回结果指定一个自定义的key：
```
$roles = DB::table('roles')->lists('title', 'name');
```

#### 指定查询语句
```
$users = DB::table('users')->select('name', 'email')->get();

$users = DB::table('users')->distinct()->get();

$users = DB::table('users')->select('name as user_name')->get();
```

#### 当前Query语句添加查询语句
```
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```
#### 使用where条件
```
$users = DB::table('users')->where('votes', '>', 100)->get();
```

#### or语法
```
$users = DB::table('users')
                ->where('votes', '>', 100)
                ->orWhere('name', 'John')
                ->get();
```

#### where: between
```
$users = DB::table('users')
                ->whereBetween('votes', array(1, 100))->get();
```

#### where: not between
```
$users = DB::table('users')
                ->whereNotBetween('votes', array(1, 100))->get();
```

#### where:in
```
$users = DB::table('users')
                ->whereIn('id', array(1, 2, 3))->get();

$users = DB::table('users')
                ->whereNotIn('id', array(1, 2, 3))->get();
```

#### where:null
```
$users = DB::table('users')
                    ->whereNull('updated_at')->get();
```

#### order by, group by  和 having
```
$users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->groupBy('count')
                    ->having('count', '>', 100)
                    ->get();
```

#### offset & Limit
```
$users = DB::table('users')->skip(10)->take(5)->get();
```

### 联接（Joins）
查询生成器也可以写join语法，看下面的例子：

#### 基本的join语法
```
DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.id', 'contacts.phone', 'orders.price')
            ->get();
```

#### left join语法
```
DB::table('users')
        ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
        ->get();
```

你可以使用更高级的join语句：
```
DB::table('users')
        ->jon('contacts', function($join)
        {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
```
若你喜欢`where`风格来处理joins,你可以在join上使用`where`和`orWhere`方法。这些方法不是比较两列，而是某列与值的比较：
```
DB::table('users')
        ->join('contacts', function($join)
        {
            $join->on('users.id', '=', 'contacts.user_id')
                    ->where('contacts.user_id', '>', 5);
        })
        ->get();
```

#### where的高级用法

#### 参数化映射
当你需要构建高级的where语句时，如：`where exists`或者嵌套的参数组，查询生成器也可以处理它：
```
DB::table('users')
        ->where('name', '=', 'John')
        ->orWhere(function($query)
        {
            $query->where('votes', '>', 100)
                  ->where('title', '<>', 'Admin')
        })
        ->get();
```
上面的query会产生如下sql:
```
select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
```

#### Exits语法
```
DB::table('users')
            ->whereExists(function($query)
            {
                $query->select(DB::raw(1))
                      ->from('orders')
                      ->whereRaw('orders.user_id = users.id');
            })
            ->get();
```
上面的query会产生如下sql:
```
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
```

### 聚合
查询生成器也有聚合的方法，如：`count`/`max`/`min`/`avg`/`sum`.

#### 使用聚合方法：
```
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');

$price = DB::table('orders')->min('price');

$price = DB::table('orders')->avg('price');

$total = DB::table('orders')->sum('price');
```

### 原生(Raw)表达式
有时候你需要用原生表达式，这些表达式会作为字串来注入query，所以你需要特别小心sql注入。使用`DB::raw`方法：

#### 使用原生表达式
```
$users = DB::table('users')
            ->select(DB::raw('count(*) as user_count, status'))
            ->where('status', '<>', 1)
            ->groupBy('status')
            ->get();
```

### 插入（Inserts）

#### 插入一条记录
```php
DB::table('users')->insert(
    array('email' => 'john@example.com', 'votes' => 0);
);
```

#### 插入一条记录并获取自增id
```php
$id = DB::table('users')->insertGetId(
    array('email' => 'john@example.com', 'votes' => 0);
);
```
>* 注意若使用的是PostgreSQL，自增列的名称限定为id，其它数据库不限定名称。

#### 插入多条记录
```php
DB::table('users')->insert(
    array('email' => 'taylor@example.com', 'votes' => 0),
    array('email' => 'daylee@example.com', 'votes' => 0)
);
```


### 更新（Updates）

#### 更新一条记录
```php
DB::table('users')
    ->where('id', 1)
    ->update(array('votes' => 1));
```

#### 某列自增或自减指定值
```php
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
```

#### 你也可以同时指定其它列的更新
```php
DB::table('users')->increment('votes', 1, array('name' => 'John'));
```


### 删除（Deletes）

#### 删除表指定记录
```php
DB::table('users')->where('votes', '<', 100)->delete();
```

#### 删除表所有记录
```php
DB::table('users')->delete();
```

#### truncate表
```php
DB::table('users')->truncate();
```


### 联合（Unions）
查询生成器提供了一个快捷的方式来`union`来个query:

```php
$first = DB::table('users')->whereNull('first_name');

$users = DB::table('users')->whereNull('last_name')->union($first)->get();

```
`unionAll`方法也可以使用，跟`union`使用方法一样。

### 悲观锁
查询生成器提供一些功能帮助你处理SELECT语句。

若需要执行`shared lock`，你需要在query上使用`sharedLock`方法：
```
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
```

为了给SELECT语句添加更新锁，你需要在query上使用`lockForUpdate`方法：
```
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```


### 缓存Queries
使用`remember`方法可以很容易的缓存query的结果
```
$users = DB::table('users')->remember(10)->get();
```
上例中，query的结果会被缓存10分钟。在缓存有效期内，这个query不会查询数据库，它会从你应用指定的缓存驱动器上加载数据。

如果你在使用支持的缓存驱动器，你可以给缓存添加标签：

```
$users = DB::table('users')->cacheTags(array('people', 'authors'))->remember(10)->get();
```
