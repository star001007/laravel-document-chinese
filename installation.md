# 安装

### 安装composer
Laravel使用composer来管理依赖。首先，下载一份`composer.phar`,安装后，你可以在当前项目目录中使用或者移动到`/usr/local/bin`作为全局命令来使用。在windows里，你可以使用[composer安装器](https://getcomposer.org/Composer-Setup.exe)。

### 安装Laravel

#### 通过Laravel安装器
首先，通过composer下载Laravel安装器：
```
composer global require "laravel/installer=~1.1"
```
确保`~/.composer/vendor/bin`在你的PATH环境变量里，使Laravel执行时可以找到。

一旦安装成功后，简单的使用命令：`laravel new `就可以在你指定的目录创建一个全新的laravel。比如，`laravel new blog`会创建一个目录为blog的并包含所有依赖的laravel。这种安装相比较通过composer安装时非常快捷的。 

#### 通过composer create-project 命令
使用composer的`create-project`命令：
```
composer create-project laravel/laravel --prefer-dist
```

#### 直接下载
安装composer后，下载最新版本的laravel框架并解压。接下来，在你laravel应用的根目录，执行`php composer.phar install` 或者 `composer install`命令来安装框架的依赖。为了顺利完成安装，这个过程还需要你有安装git。


### 服务器环境的要求
laravel框架需要如下的系统要求：
>* PHP版本>=5.4
>* PHP的Mcrypt扩展已安装

PHP5.5以上的版本，一些操作系统可能还需要你安装JSON扩展。当你使用Ubuntu时，你可以执行`apt-get install php5-json`来完成。

### 配置
完成安装Laravel后的第一件事情就是将你的应用key设置为一个随机的字串。如果你通过composer来安装的laravel，这个key可能已经通过`key:generate`命令被设置好了。通常情况下，这个字串的长度是32个字符。它是在`app.php`配置文件中可以进行配置。如果应用的key没有被设置，那么你的应用的用户session及其他的编码的数据的安全性将会受到威胁。

Laravel几乎不需要怎么配置，就可以欢快的开始编码啦。然而，你可能还想再看下`app/config/app.php`文件及相关文档。它包含了一些选项比如：`timezone`和`locale`可能根据您的应用需要进行更改。

当laravel安装后，你同样也需要配置下本地环境，在本地进行测试时候来获取更详细的错误信息。默认地，在生产环境的配置文件里详细的错误报告是被禁用的。
>注意：在生产环境下，千万别把`app.debug`设置为true.

#### 权限
laravel需要一些权限配置，`app/storage`目录需要web服务器的写操作的权限。

#### 路径
框架的几个目录路径是可以配置的。想改变这些目录的位置，请查看文件`bootstrap/paths.php`。
### 优美的URLs

#### Apache
框架的`public/.htaccess`文件可以设置url去除index.php。如果你使用的web服务器是Apache，确保你开启了`mod_rewrite`模块。
如果这个`.htacess`文件在Apache里不起作用，试下如下代码：
```apache
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]

```

#### Nginx
对于Nginx，使用下列的设置：
```nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}

```

