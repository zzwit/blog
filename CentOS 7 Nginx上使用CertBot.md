# CentOS 7 Nginx上使用CertBot 
**简介**

Let's Encrypt是一种新的证书颁发机构（CA），它提供了一种获取和安装免费TLS / SSL证书的简单方法，从而在Web服务器上启用加密的HTTPS。它通过提供软件客户端Certbot来简化流程，该软件客户端试图使大多数（如果不是全部的话）所需步骤自动化。目前，获取和安装证书的整个过程在Apache和Nginx Web服务器上都是完全自动的。

在本教程中，我们将向您展示如何使用certbotLet's Encrypt客户端获取免费的SSL证书，并在CentOS 7上将其与Nginx一起使用。我们还将向您展示如何自动更新您的SSL证书。

**先决条件**

在学习本教程之前，您需要做一些事情。

*   拥有sudo特权的非root用户的CentOS 7服务器。您可以按照CentOS 7教程的初始服务器设置中的步骤1-3来学习如何设置这样的用户帐户。
*   您必须拥有或控制您希望使用该证书的注册域名。如果您还没有注册域名，您可以在其中注册一个域名注册商（例如Namecheap，GoDaddy等）。
*   DNS A Record将您的域名指向您的服务器的公共IP地址。这是必需的，因为让我们加密验证您拥有它颁发证书的域。例如，如果您想获取证书example.com，那么该域必须解析到您的服务器才能使验证过程正常工作。我们的设置将使用example.com和www.example.com作为域名，所以这**两个DNS记录都是必需的**。
*   一旦完成了所有先决条件，让我们继续安装Let's Encrypt客户端软件。

## 第1步 - 安装Certbot让我们加密客户端
使用Let's Encrypt获取SSL证书的第一步是certbot在您的服务器上安装该软件。目前，最好的安装方式是通过EPEL存储库。

输入以下命令以启用对服务器上EPEL存储库的访问：

```
$ sudo yum install epel-release

```
储存库启用后，您可以certbot-nginx通过键入以下命令来获取该软件包：

```
$ sudo yum install certbot-nginx

```

该certbot让我们加密客户端已经安装，并准备使用。

## 第二步安装Nginx（如果安装过的请忽略）
如果你还没有安装Nginx，你现在可以这样做。EPEL存储库应该已经在前一节中启用，所以你可以通过输入以下命令来安装Nginx：

```
$ sudo yum install nginx
```

然后，使用systemctl以下命令启动Nginx ：

```
$ sudo systemctl start nginx
```
Certbot可以自动为Nginx配置SSL，但它需要能够server在您的配置中找到正确的块。它通过查找server_name与您要申请证书的域匹配的指令来完成此操作。如果你正在盯着一个新的Nginx安装，你可以更新默认的配置文件：

```
sudo vi /etc/nginx/nginx.conf 或者  sudo vim /usr/local/nginx/conf/nginx.conf

```
找到现有的server_name行：
找到你nginx的conf配置文件

```
server_name example.com www.example.com;

```
保存该文件并退出编辑器。使用以下内容验证配置编辑的语法：

```
sudo nginx -t

```

如果运行没有错误，请重新加载Nginx以加载新配置：

```
sudo systemctl reload nginx

```

Certbot现在将能够找到正确的server块并更新它。现在我们将更新我们的防火墙以允许HTTPS流量。

## 第3步 - 更新防火墙
如果您启用了防火墙，请确保端口80和443对传入流量开放。如果你没有运行防火墙，你可以跳过。

如果您正在运行**防火墙**，则可以通过键入以下命令来打开这些端口：

```
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=https
sudo firewall-cmd --runtime-to-permanent

```
如果运行iptables防火墙，则需要运行的命令高度依赖于当前的规则集。对于基本规则集，您可以键入以下内容来添加HTTP和HTTPS访问权限：

```
sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT

firewall-cmd --zone=public --add-port=443/tcp --permanent

```
设置完防火墙重启防火墙服务
检查端口是否开发

```
nc -vz IP地址 22
```

如果防火墙规则都已经开发端口，还是不能好使的，请查看的服务器的运营商是否用安全策略

我们现在准备运行Certbot并获取我们的证书。

## 第4步 - 获得证书

Certbot通过各种插件提供了各种获取SSL证书的方式。Nginx插件将负责重新配置Nginx并在必要时重新加载配置：
这certbot与--nginx插件一起运行，-d用于指定我们希望证书有效的名称。

```
	sudo certbot --nginx -d example.com -d www.example.com
```
运行报错这样的错误ImportError: No module named 'requests.packages.urllib3' 解决办法如下：

```
	sudo pip install --upgrade --force-reinstall 'requests==2.6.0' urllib3
```

如果提前已经安装了nginx了，vhost某service服务

```
这里我不想使用CertBot的standalone模式，这个模式虽然可以配置好服务器，但是以后Renew的时候，需要让服务停止一下，
再启动。因此抛弃这个模式，我们使用Webroot配置模式。
因为，CertBot在验证服务器域名的时候，会生成一个随机文件，然后CertBot的服务器会通过HTTP访问你的这个文件，
因此要确保你的Nginx配置好，以便可以访问到这个文件。
修改你的服务器配置，在server模块添加：

	location ^~ /.well-known/acme-challenge/ {
	   default_type "text/plain";
	   root     /usr/share/nginx/html; #项目站点目录
	}
	
	location = /.well-known/acme-challenge/ {
	   return 404;
	}
	
接着重新加载Nginx配置：
sudo service ngin reload

然后针对这个域名生产证书

sudo certbot certonly --webroot -w /home/wwwroot/wwwhpqm/ -d www.hupanqingmiao.com



```

第一次运行可能让你设置邮箱

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator webroot, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to

```

可能会出现让你同意条款

```
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
(A)gree/(C)ancel:

输入 A
```

如果成功，certbot将询问您希望如何配置HTTPS设置：

```
	Output
	Please choose whether HTTPS access is required or optional.
	-------------------------------------------------------------------------------
	1: Easy - Allow both HTTP and HTTPS access to these sites
	2: Secure - Make all requests redirect to secure HTTPS access
	-------------------------------------------------------------------------------
	Select the appropriate number [1-2] then [enter] (press 'c' to cancel):

```
选择你的选择，然后点击ENTER。配置将被更新，并且Nginx将重新加载以获取新的设置。certbot将包含一条消息，告诉您该过程已成功完成，并存储证书的位置：

```
	Output
	IMPORTANT NOTES:
	 - Congratulations! Your certificate and chain have been saved at
	   /etc/letsencrypt/live/example.com/fullchain.pem. Your cert will
	   expire on 2017-10-23. To obtain a new or tweaked version of this
	   certificate in the future, simply run certbot again with the
	   "certonly" option. To non-interactively renew *all* of your
	   certificates, run "certbot renew"
	 - Your account credentials have been saved in your Certbot
	   configuration directory at /etc/letsencrypt. You should make a
	   secure backup of this folder now. This configuration directory will
	   also contain certificates and private keys obtained by Certbot so
	   making regular backups of this folder is ideal.
	 - If you like Certbot, please consider supporting our work by:
	
	   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
	   Donating to EFF:                    https://eff.org/donate-le
```

提示上边的信息证明您的证书已下载，安装并加载。尝试重新加载您的网站https://并注意您的浏览器的安全指示器。它应该表示该网站已妥善保护

## 第5步 - 配置Nginx 设置443接口

同样，修改Nginx的虚拟主机配置文件，新建一个443端口的server配置：

```
server {
        listen 443 ssl;
        listen [::]:443 ssl ipv6only=on;

        ssl_certificate /etc/letsencrypt/live/www.test.com/fullchain.pem;
        ssl_certificate_key  /etc/letsencrypt/live/www.test.com/privkey.pem;   
        // ... other settings ...
}

```

重启nginx


```
  sudo service  nginx reload
```

## 自动更新证书
先在命令行模拟证书更新：

```
sudo crontab -e

```


你的文本编辑器将打开默认的crontab，这是一个空文本文件。粘贴在以下行中，然后保存并关闭它

```
15 3 * * * /usr/bin/certbot renew --quiet >> /home/wwwlogs/le-renew.log

```
## 参考
[Certbot官网](https://certbot.eff.org/docs/)

[How To Secure Nginx with Let's Encrypt on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7)











