# 快速开始
[TOC]

### 安装
#### 通过`Laravel installer`安装
首先，通过Composer下载 Laravel installer。
```
composer global require "laravel/installer=~1.1"
```
确保`~/.composer/vender/bin`目录在PATH存在，使得你在执行 **Laravel**命令时，可以找到laravel可执行文件。

一旦安装后，简单的命令 `laravel new `命令可以在你指定的位置创建一个全新的laravel。例如，`laravel new blog `会在当前路径下创建一个名称为**blog**的文件夹，包含了全新的laravel及所有的依赖。这种安装方法比通过`Composer`安装更快捷。


#### 通过`Composer`安装
Laravel框架使用 [Composer](https://getcomposer.org/) 来安装和依赖管理。如果你还没有安装，从 [安装Composer](https://getcomposer.org/doc/00-intro.md) 开始吧。
接下来你可以在终端打出下列命令来安装Laravel：
```
composer create-project laravel/laravel your-project-name --prefer-dist
```
这条命令将会在当前目录下的`your-project-name`文件夹下载并安装一个全新的laravel。

如果你愿意，你还可以从Githup的[Laravel仓库](https://github.com/laravel/laravel/archive/master.zip)来下载，接着在你创建的项目的根目录下运行`composer install`命令，这条命名会下载并安装框架的依赖。

#### 权限（Permissions）
安装laravel后，你可能需要给web server在 `app/storage`目录上写文件的权限。更多配置的详细信息看[安装文档](http://laravel.com/docs/4.2/installation)

#### Laravel服务器
通常情况下，你可能会用`Apache`或者`Nginx`类似的服务器来作为您的Laravel应用的服务器。如果您使用的PHP5.4版本以上，可能会想用PHP内置的开发服务器，你可以使用 **Artisan命令** `server`：
```
php artisan serve
```

#### 目录结构

安装框架后，让我们熟悉下目录结构。`app`目录包含文件夹：`views`,`controllers`和`models`，您的应用的大部分的代码都在这些目录里。你可能还想看下`app/config`目录及相关的配置选项。


### 本地开发环境
过去，配置一个本地的PHP开发环境是一件令人头疼的事情。安装合适版本的PHP、必须的扩展以及其他一些部件是一件困惑、耗时的事情。代替使用[Laravel Homestead](http://laravel.com/docs/4.2/homestead)，Homestead是为Laravle和Vagrant设计的简单的虚拟机。因为Homestead vagrant box 已经提前预装好了您构建一个健壮PHP应用的所有软件，你可以在几秒钟内就创建一个虚拟的、隔离的开发环境。下面是Homestead包含的一些软件：
>* Nginx
>* PHP 5.6
>* MySQL
>* Redis
>* Memcached
>* Beanstalk

不要担心，即使“虚拟”听起来也挺复杂的。`VirtualBox` 和 `Vagrant`,是`Homestead`的两个依赖，包含适用于几乎所有开源系统的简单的、图形化的安装器 。更多内容查看[Homestead 文档](http://laravel.com/docs/4.2/homestead)。

### 路由
开始前，让我们创建第一个路由器。在Laravel里，最简单的路由是一个闭包。打开`app/routes.php`文件，在文件末尾添加下列路由
```php
Route:get('users', function()
{
    return 'Users!';
});

```
现在，如果你在web浏览器里输入`/users`路由，你将会看到响应是：Users! 太棒了，你刚刚创建一个第一个路由。

路由还可以绑定到控制器方法上。比如：
```php
Route::get('users', 'UserController@getIndex');
```
这个路由告知框架请求`/users`路由会调用 `UserController`控制器类的 `getIndex`方法。了解更多的控制器路由，查看[控制器文档](http://laravel.com/docs/4.2/controllers)。

### 创建一个视图
接下来，我们创建一个简单视图来展示user数据。视图模板都在`app/views`目录里，宝航了您应用的HTML。接着我们在当前目录下创建两个视图：`layout.blade.php` 和 `users.blade.php`。首先，让我们创建 `layout.blade.php`文件：
```
<html>
    <body>
        <h1>Laravel Quickstart</h1>
        @yield('content')
    </body>
</html>
```
接下来，我们创建 `users.blade.php`:
```
@extends('layout')

@section('content')
    Users!
@stop
```
其中的一些语法看起来相当的奇怪。这是因为我们使用了Laravel模板系统：`Blade`.Blade 非常快，因为它仅仅在模板系统上使用了极少的正则表达式来编译成纯粹的PHP代码。Blade提供了强大的功能，如模板继承，以及类似PHP控制结构的if for语法糖。查看更多信息，请看[Blade文档](http://laravel.com/docs/4.2/templates)。

现在，我们已经有了两个模板，让我们从`/users`的路由中返回，而不再是 `Users!`，路由代码如下：
```
Route::get('users', function()
{
    return View::make('users');
});
```
太棒了，现在我们创建了一个继承layout的视图。接下来，让我们开始使用数据库层。

### 创建一个迁移

为了创建一个拥有数据的表，我们需要使用Laravel的迁移系统。迁移允许你丰富地定义对数据库的修改，并很容易将修改对您的团队进行分享。

首先，让我们配置一个数据库连接。你可以在 `app/config/database.php`文件里配置你所有的数据库连接。 默认地，Laravel使用MySQL驱动器，你需要在数据库配置文件里提供验证信息。 如果你愿意，可以将驱动器选项改为 `sqlite`,那么它就会使用包含在 `app/database`目录里的SQLite数据库。

接下来，为了创建一个迁移。我们将使用`Aitisan CLI` 。在项目的根目录，在终端执行下列代码：
```
php artisan migrate:make create_users_table
```
接下来，知道在`app/database/migrations`文件夹里生成的迁移文件。这个文件包含了一个类有两个方法：up和down。在up方法里，你可以进行对数据表的更改。在down方法里，你仅需要颠倒它们。

```php
public function up()
{
    Schema::create('users', function($table)
    {
        $table->increments(id);
        $table->string('email')->unique();
        $table->string('name');
        $table->timestamps();
    });
}

public function down()
{
    Schema::drop('users');
}

```
最后，我们可以在终端运行`migrate`命令来进行迁移操作。注意：在项目的根目录里进行执行。
```
php artisan migrate

```

如果你想回滚您的迁移，使用`migrate:rollback`命令。现在我们已经有了一个数据表，让我们开始添加一些数据吧。

### Eloquent ORM

Laravel 配备一个强大的ORM：Eloquent。如果你使用过Ruby on Rails 框架，那么将会对Eloquent非常熟悉，它也遵循了 ActiveRecord 数据库交互的风格。

首先，让我们建一个模型：一个Eloquent模型可以用来查询关联数据表，会返回数据表中指定的行。不要担心，很快会弄清楚的。模型通常存放在`app/models`目录下。让我们在这个目录下定义一个`User.php`模型如下：
```
class User extends Eloquent{}
```
注意，我们不需要告诉Eloquent我们使用哪儿个数据表。Eloquent有大量的便捷操作，其中一个就是使用模型的复数形式作为模型数据库的数据表。非常方便有木有?

使用数据库管理工具在你的users数据表里插入几行数据，我们接下来用Eloquent来获取它们并传递到我们的视图。

现在，让我们修改我们`/users`路由如下：
```
Route::get('users', function(){
    $users = User::all();
    return View::make('users')->with('users', $users);
});
```
我们一行行的看下上面的路由。首先，模型`Users`的`all`方法会获取数据表users的所有的行。接下里，我们通过`with`方法来传递这些结果行到View视图。这个`with`方法接收两个参数：key和value，以在视图中使用。

太棒了，现在我们已经准备好在我们的视图中展示`users`表的数据了。

### 展示数据
现在让我们使用上面的`$users`变量，我们可以像下面这么做：
```
@extends('layout')

@section('content')
    @foreach($users as $user)
        <p>{{ $user->name }}</p>
    @endforeach
@stop
```
你可能想问，我们的`echo`在哪儿里呢？其实 当我们使用`Blade`时，你输出的数据需要用两个大括号来引起来。这个很简单，现在我们可以再浏览器中输入`/users`路由，那么将会看到结果中会有我们的users信息。

这仅仅是个开始。在这个教程里，你只看到了Laravel的最基本的东西，但是还有很多激动人心的事情等着我们去发掘。阅读 `Eloquent` 和 `Blade`的详细文档，会让你发现它的强大特性。当然，你也可能会对 `队列：Queues` 和 `单元测试：Unit Testing` 更感兴趣。 话说回来，如果你想通过 Ioc容器来更改你的架构， 这一切都由你来做主。
### 部署你的应用
Laravel的其中一个目标就是让PHP应用开发从下载到部署都是简单快乐的， [Laravel Forge](https://forge.laravel.com) 就可以让您的Laravel应用很简单的部署到超级快的服务器上面。 Forge 可以配置、提供DigitalOcean/Linode/Amazon EC2 服务器。如Homestead一样，所有最新的软件如：Nginx/PHP 5.6/MySQL/Postgres/Redis/Memcached 等都已经被安装好了。Forge的**快速部署[Quick Deploy]**甚至可以在你每次给Githup或Bitbucket提交修改时，同时将你的代码就行同步更新。

除了这些，Forge可以帮助你配置`队列进程:queue workers`/`SSL`/`Cron jobs`/`sub-domains`等，更多信息，请查看[Forge website](https://forge.laravel.com)
