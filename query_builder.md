# 基本用法

### 配置
laravel配置数据库、执行`queries`非常简单。配置文件在`app/config/database.php`里，你可以定义定义所有的数据库连接及默认地数据库连接。范例中包含了了所有支持的数据库连接。

目前之前的数据库有：MySQL/Postgres/SQLite/SQL Searver。

### 读/写连接
有时候我们需要在`SELECT`语句时选择一个数据库连接，而在`INSERT`,`UPDATE`,`DELETE`语句时选择其他的数据库连接，laravel可以非常简单处理，不管你用的是
原生queries，查询生成器还是Eloquent ORM。

如何配置读/写连接，看下面的例子：
```php
'mysql' => array(
    'read' => array(
        'host' => '192.168.1.1',
    ),
    'write' => array(
        'host' => '196.168.1.2'
    ),
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
),
```
注意我们添加了两个keys：`read`和`write`。它们共享mysql里所有的公共配置，需要自定义的选项可以添加到`read`和`write`对应的数组里。

### 执行Queries
数据库连接配置成功后，你就可以如下使用`DB`类来执行queries。

#### 执行Select查询
```php
$results = DB::select('select * from users where id = ?', array(1));
```
select方法会返回结果集数组。

#### 执行Insert语句
```php
DB::insert('insert into users (id, name) values (?, ?)', array(1, 'Dayle'));
```

#### 执行Update语句
```php
DB::update('update users set votes = 100 where name = ?', array('John'));
```

#### 执行Delete语句
```php
DB::delete('delete from users');
```

>* 注意：update和delete语句会返回操作的影响行数。

#### 执行一般语句
```php
DB::statement('drop table users');
```

#### 监听所有Query的事件
使用`DB::listen`方法：
```php
DB::listen(function($sql, $bindings, $time)
{
    //
});
```

### 事务
为执行一系列数据库事务的操作，你需要使用`transaction`方法：
```php
DB::transaction(function()
{
    DB::table('users')->update(array('votes' => 1));

    DB::table('posts')->delete();
});
```
>* 注意：`transaction`闭包里抛出的异常会使得这个事务在后台自动回滚。

若需手动启动一个事务：`beginTransaction`
```php
DB::beginTransaction();
```
回滚：`rollback`
```php
DB::rollback();
```
提交：`commit`
```php
DB::commit();
```

### 获取数据库连接
如果你有多个连接时，可以通过`DB::connection`方法来获取：
```
$users = DB::connection('foo')->select(...);
```
也可以获取原生部署的pdo实例：
```
$pdo = DB::connection->getPdo();
```
若数据库连接超过了pdo实例的`max_connections`限制，那么可以使用`disconnect`方法来断开：
```
DB::disconnect('foo');

```

### Query日志

默认地，laravel会讲当前请求的执行的所有queries放在内存里。但是，当我们的插入大量的数据时，可能会占用过多的内存。那么，你可以使用`disableQueryLog`来禁用它：
```
DB::connection()->disableQueryLog();
```

若想获取执行的queries，使用`getQueryLog`方法：
```php
$queries = DB::getQueryLog();
```