# mac 安装opensll 最新版本
### 第一步：先更新brew

```
> brew update
```

		
### 安装最新版本

```
> brew install openssl 

提示

Warning: openssl 1.0.2o_1 is already installed and up-to-date
To reinstall 1.0.2o_1, run `brew reinstall openssl`

执行 > brew reinstall openssl

安装成功
```

查看版本

```
> openssl version
> OpenSSL 0.9.8zh 14 Jan 2016
	
```
还是原来的版本，差看一下安装目录吧

```
> which openssl
> usr/bin/openssl
> ll /usr/local/Cellar/openssl
> 1.0.2l
  1.0.2m
  1.0.2n
  1.0.2o_1
```
发现有很多版本，于是我选择了和我php安装的openssl 版本一至 '1.0.2n'，现在要将原来的版本替换新的版本

```
设置软链
> ln -s /usr/local/Cellar/openssl/1.0.2n/bin/openssl /usr/local/bin/openssl
将原来的 /usr/bin/openssl 重新命名
> sudo  mv /usr/bin/openssl /usr/bin/openssl_OLD
mac 升级的到osx 升级到 10.11 导致的 很多重要文件都受限了
> 提示mv: rename /usr/bin/openssl to /usr/bin/openssl_OLD: Operation not permitted
```

为了在启用了[系统完整性保护的情况下](https://developer.apple.com/library/content/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html)在macOS Sierra上解决[OCSP状态请求扩展无限制内存增长（CVE-2016-6304）](https://www.openssl.org/news/secadv/20160922.txt)：

[OSX 10.11关闭SIP方法](https://jingyan.baidu.com/article/e5c39bf5d13bf939d76033cf.html)

重新链接最新版本：

```
> sudo ln -s /usr/local/Cellar/openssl/1.0.2n/bin/openssl /usr/bin/openssl
```
恢复原始权限/usr/local/bin

```
> sudo chown root:wheel /usr/local
```



	
	
