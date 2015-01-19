# 版本说明

### Laravel 4.2版本
你可以通过命令：`php artisan changes`来查看这次版本的所有变化，或者可以通过[Githup文件](https://github.com/laravel/framework/blob/4.2/src/Illuminate/Foundation/changes.json)。这些说明涵盖了所有的主要改进及变化。

>注意：在4.2版本里，许多小的bug修复及改进都体现到了4.1版本中。所以，你可以同时看下4.1版本的更改列表。

#### 需要PHP5.4+版本以上支持
Laravel4.2需要PHP5.4或者更改的版本。为了使用PHP的新特性如`traits`，以提供更具表现力的接口如：laravel Cashier。PHP5.4相比5.3同样带来了速度跟性能上的提升。

#### Laravel Forge
Laravel Forge是全新的基于web的应用，使得在云上，如Linode, DigitalOcean, Rackspace, and Amazon EC2 创建和管理PHP服务器变得非常简单。支持自动化的配置，SSH密钥的访问，后台任务的自动化，NewRelic & Papertrail的服务器监控，“更改即部署”，Laravel进程队列配置等等。Forge提供所有Laravel应用运行的最简单、最经济实惠的办法。

Laravel4.2的配置文件`app/config/databases.php`默认使用了Forge,以更快捷的来进行部署。

更多关于Laravel Forge的信息，查看 [Forge官方网站](https://forge.laravel.com)

#### Laravel Homestead
Laravel Homestead 是官方的vagrant环境用来部署健壮的Laravel和PHP应用。The vast majority of the boxes' provisioning needs are handled before the box is packaged for distribution, allowing the box to boot extremely quickly. Homestead includes Nginx 1.6, PHP 5.6, MySQL, Postgres, Redis, Memcached, Beanstalk, Node, Gulp, Grunt, & Bower. Homestead includes a simple Homestead.yaml configuration file for managing multiple Laravel applications on a single box.

The default Laravel 4.2 installation now includes an app/config/local/database.php configuration file that is configured to use the Homestead database out of the box, making Laravel initial installation and configuration more convenient.

The official documentation has also been updated to include Homestead documentation.


#### Laravel Cashier
Laravel Cashier is a simple, expressive library for managing subscription billing with Stripe. With the introduction of Laravel 4.2, we are including Cashier documentation along with the main Laravel documentation, though installation of the component itself is still optional. This release of Cashier brings numerous bug fixes, multi-currency support, and compatibility with the latest Stripe API.

#### Daeamon Queue Workers
The Artisan queue:work command now supports a --daemon option to start a worker in "daemon mode", meaning the worker will continue to process jobs without ever re-booting the framework. This results in a significant reduction in CPU usage at the cost of a slightly more complex application deployment process.

More information about daemon queue workers can be found in the queue documentation.

#### 邮件API驱动
Laravel 4.2 introduces new Mailgun and Mandrill API drivers for the Mail functions. For many applications, this provides a faster and more reliable method of sending e-mails than the SMTP options. The new drivers utilize the Guzzle 4 HTTP library.

#### 软删除接口
A much cleaner architecture for "soft deletes" and other "global scopes" has been introduced via PHP 5.4 traits. This new architecture allows for the easier construction of similar global traits, and a cleaner separation of concerns within the framework itself.

More information on the new SoftDeletingTrait may be found in the Eloquent documentation.
#### 方便的权限 & Remindable Traits
The default Laravel 4.2 installation now uses simple traits for including the needed properties for the authentication and password reminder user interfaces. This provides a much cleaner default User model file out of the box.
#### 简单分页
A new simplePaginate method was added to the query and Eloquent builder which allows for more efficient queries when using simple "Next" and "Previous" links in your pagination view.
#### 迁移确认
In production, destructive migration operations will now ask for confirmation. Commands may be forced to run without any prompts using the --force command.
### Laravel 4.1版本

#### 全部变化

#### 新的SSH组件
































#### 