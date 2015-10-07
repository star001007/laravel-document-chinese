# Homestead

### 介绍
laravle 始终贯彻让php的开发体验更愉悦的理念，包括本地开发环境。`Vagrant`提供了一个简易、优雅的方法去管理和提供虚拟主机。

laravel Homestead 是一个官方的，预装Vagrant的#盒子# ，它提供了一个非常棒的环境不需要你在本机上安装php/hhvm/web服务器/其他服务软件。不用担心会弄乱你的操作系统，vagrant盒子是可完全开放的。如果过程中出错了，你可以在几分钟内在重建一个盒子。

homestead可以在任何的windows/mac/linux系统上运行，包括了nginx服务器，php5.6，MySQL, Postgres，Redis，Memcached以及其他一些好东西，让你开发你伟大的laravel应用。
    
    注意：如果你在使用Windows，你可能需要开发hardware virtualization(vt-x)。它一般可以通过BIOS来开启。

Homestead现在用vagrant1.6来构建和测试。

### 包含的软件
```
Ubuntu 14.04
PHP 5.6
HHVM
Nginx
MySQL
Postgres
Node（集成了Bower，Grunt和Gulp）
Redis
Memcached
Beanstalkd
Laravel Envoy
Fabric + HipChat Extension
```

### 安装和设置
#### 安装VirtualBox 和Vagrant
在运行你的Homestead环境之前，你需要安装VirtualBox和Vagrant。这些软件包提供了所有流行操作系统的可视化的安装器。

#### 添加Vagrant盒子
VirtualBox和Vagrant安装完毕后，就可以使用下列的命令来添加`laravel/homestead`盒子到你的Vagrant安装。
```
vagrant box add laravel/homestead
```
需要几分钟来下载这个盒子，根据你的网络环境花费时间不同。
如果失败了，可能是你的vagrant版本问题，它需要盒子的url地址，你需要如下操作：
   
   vagrant box add laravel/homestead https://atlas.hashicorp.com/laravel/boxes/homestead

#### 安装Homestead

##### 通过Comoposer和PHP工具
一旦盒子已经添加到你的Vagrant安装，你就可以通过Composer的`global`命令来安装Homestead 命令行工具。
    
    composer global require "laravel/homestead=~2.0"
确保`~/.composer/vendor/bin`目录在你的环境变量，当执行`homestead`时，才能找到。

安装完Homestead命令行工具后，运行`init`命令来创建`Homestead.yaml`配置文件：
    
    homestead init

会在`~/.homestead`目录下生成文件`Homestead.yaml`，Mac或者Linux系统，你可以通过命令`homestead edit`来编辑这个文件。
    
    homestead edit

##### 通过Git手动安装 (没有本地PHP)
相应地，如果你不想在本机上安装PHP，你可以通过克隆版本来手动安装Homestead。考虑到克隆一个版本到所有laravel项目所在的目录作为`Homestead`目录，这样Homestead盒子将作为主机来服务所有的laravel项目：
    
    git clone https://github.com/laravel/homestead.git Homestead
安装完毕后，运行`bash init.sh`命令来创建`Homestead.yaml`配置文件：
    
    bash init.sh
创建的`Homestead.yaml`文件在`~/.homestead`目录下。

#### 设置你的SSH Key
接下里，你需要编辑`Homestead.yaml`文件。这个文件里，你可以配置你的公钥的路径，以及主机跟Homestead虚拟机共享目录。

若没有SSH Key，Mac或者Linux系统，你可以通过下列命令生产SSH key 对：
    
    ssh-keygen -t rsa -C "you@hoemstead"

若是Windows，你可以按照Git 通过使用Git自带的`Git Bash`shell来执行上面的命令。若没有，你也可以用`PuTTY`和`PuTTYgen`。

创建好SSH Key之后，指定配置文件`Homestead.yaml`中`authorize`属性为key的路径。

#### 配置共享文件夹
`Homestead.yaml`文件中的属性`folders`列出所有你想与Homestead环境共享的文件夹。文件夹中的文件发生改变，会在本机跟Homestead环境中保持同步，你可以根据需要来指定任意数量的文件夹。

#### 配置你的Nginx站点
如果你之前对Nginx不熟悉，没关系。属性`sites`使得你很容易的把一个域名跟Homestead环境中的文件夹映射好。`Homestead.yaml`文件中包含了一个简单站点的配置。当然，你可以根据需要来给你的Homestead环境添加任意站点。Homestead服务于你运行每个laravel项目，是一个非常简洁的虚拟化环境。

你可以让Homestead站点支持`HHVM`，只需要设置`hhvm`选项的值为`true`:
```
sites:
    - map: homestead.app
      to: /home/vagrant/Code/laravel/public
      hhvm: true      
```


#### Bash别名
需要给Homestead添加bash别名的话，只需要添加到`~/.homestead`目录的`aliases`文件中。


#### 运行Vagrant盒子
根据你的需要编辑过`Homestead.yaml`后，运行`homestead up`命令。如果你是手动安装的homestead，而不是使用的PHP`homestead`工具，在git克隆的homestead的文件夹里，运行`vagrant up`命令。

vagrant会运行虚拟机，并且自动配置你的共享文件夹和Nginx站点。想删除的话，使用`homestead destroy`命令。查看所有可用的homestead命令，运行`homestead list`。

不要忘记在`host`文件里添加nginx站点的域名！host文件会讲你本地的域名指向安装的homestead环境。Mac和linux，这个文件在`/etc/hosts`。对于windows，在`C:\Windows\System32\drivers\etc\hosts`。你需要添加类似下面的记录：
    
    192.168.10.10 homestead.app

确保你的ip地址是你在`Homestead.yaml`中配置的地址。`hosts`文件中添加后，你可以通过浏览器访问这个站点。
    
    http://homestead.app

下面会学习如何连接数据库！

### 日常使用

#### 通过ssh连接homestead
若想通过ssh连接homestead环境，只需要在终端命令行中输入`homestead ssh` 或者`vagrant ssh`就可以登录。

#### 连接数据库
`homestead`数据库配置成mysql或者postgres。为了更方便，laravel本地的数据配置被设置成默认使用这个数据库。
需要连接的话，可以通过你主机的navicat或者sequel pro连接，连接主机是`127.0.0.1`，端口是`33060`
(mysql)，`54320`(postgres)。用户名密码是`homestead/secret`。
    
    注意：你只能用这些非标准的短裤来连接数据库。默认的3306/5432端口是laravel数据库配置文件中指定需要使用的，因为laravel是运行在虚拟机上的。

#### 添加其他站点
homestead环境安装好并且运行后，你可以根据需要添加其他站点，运行任意个laravel在一个虚拟的homestead环境中。首先：在`homestead.yaml`文件中添加站点，接着，运行`vagrant provision`。

同时，你也可以使用homestead环境中可用的`serve`脚本。想使用`serve`脚本，ssh登入homestead环境，接着运行下面命令：
    
    serve domain.app /home/vagrant/Code/path/to/public/directory

    注意：运行`serve`命令后，不要忘记在`hosts`中添加域名。

### 端口

下面端口是指向你的homestead环境：

```
ssh:2222 -> forwards to 22
http:8000 -> forwards to 80
mysql:33060 -> forwards to 3306
postgre:54320 -> forwards to 5432

```















