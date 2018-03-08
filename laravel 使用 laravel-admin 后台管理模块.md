# laravel 使用 laravel-admin 后台管理模块
	1. laravel > 5.4
	2. laravel-Laraadmin "1.5.*"
	3. PHP >= 7.0
	
### 参考地址
   * [laravel框架5.5文档](https://docs.golaravel.com/docs/5.5/installation)
   * [laravel-Laraadmin中文文档](http://laravel-admin.org/docs/#/zh/installation)
   * [laravel-Laraadmin演示地址](http://laravel-admin.org/demo)
   
## 1. 安装laravel框架 [地址](https://docs.golaravel.com/docs/5.5/installation/)
	安装laravel版本要求
	PHP >= 7.0.0
	OpenSSL PHP Extension
	PDO PHP Extension
	Mbstring PHP Extension
	Tokenizer PHP Extension
	XML PHP Extension
### 使用composer 安装laravel
	1.composer create-project --prefer-dist laravel/laravel lzm_admin
	2. cd lzm_admin
	3. sudo chmod -R 777 storage/ bootstrap/ database/migrations/

## 安装LaraAdmin软件包 [1.5](http://laravel-admin.org/docs/#/zh/installation?id=%e5%ae%89%e8%a3%85)
当前版本(1.5)需要安装PHP 7+和Laravel 5.5, 如果你使用更早的版本，请参考文档: [1.4](http://laravel-admin.org/docs/v1.4/#/zh/)
	
```
  composer require encore/laravel-admin "1.5.*"
```
	
然后运行下面的命令来发布资源：

```
php artisan vendor:publish --provider="Encore\Admin\AdminServiceProvider"

```	

在该命令会生成配置文件config/admin.php，可以在里面修改安装的地址、数据库连接、以及表名，建议都是用默认配置不修改。

然后运行下面的命令完成安装：

```
 php artisan admin:install

```
启动服务后，在浏览器打开 <font color=#f6712e>http://localhost/admin/ </font>,使用用户名 <font color=#f6712e>admin</font> 和密码 <font color=#f6712e>admin</font>登陆.

## 生成的文件
安装完成之后,会在项目目录中生成以下的文件:

## 配置文件
安装完成之后，laravel-admin所有的配置都在config/admin.php文件中。

## 后台项目文件

安装完成之后，后台的安装目录为app/Admin，之后大部分的后台开发编码工作都是在这个目录下进行。

```
app/Admin
├── Controllers
│   ├── ExampleController.php
│   └── HomeController.php
├── bootstrap.php
└── routes.php

```

<font color=#f6712e>app/Admin/routes.php</font> 文件用来配置后台路由。

<font color=#f6712e>app/Admin/bootstrap.php</font> 是laravel-admin的启动文件, 使用方法请参考文件里面的注释.

<font color=#f6712e>app/Admin/Controllers</font> 目录用来存放后台控制器文件，该目录下的HomeController.php文件是后台首页的显示控制器，ExampleController.php为实例文件。

## 静态文件

后台所需的前端静态文件在<font color=#f6712e>/public/vendor/laravel-admin</font>目录下.

## 安装一些细节的配置
### 设置语言类型
完成安装之后，默认语言为英文(en)，如果要使用中文，打开<font color=#f6712e>config/app.php</font>，将<font color=#f6712e>locale</font>设置为<font color=#f6712e>zh-CN</font>即可。
	
### 配置域名
 打开 <font color=#f6712e>.env</font> 文件
 
```
APP_URL=http://lv_admin.my.com
```
 
  
### 数据库配置

打开 <font color=#f6712e>.env</font> 文件

```
	DB_CONNECTION=mysql
	DB_HOST=127.0.0.1
	DB_PORT=3306
	DB_DATABASE=laravel_admin_demo
	DB_USERNAME=root
	DB_PASSWORD=root
```



### 后期会更新一些更多的在开发中注意的细节



















