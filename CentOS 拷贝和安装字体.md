# liunx 拷贝和安装字体
> 安装执行命令
> 第一步 安装字体
```
yum install fontconfig
yum install mkfontscale

安装完之后会生产 /usr/share/fonts
cd /usr/share/fonts
mkdir chinese
```
> 第二步 将ttc文件拷贝到/usr/share/fonts/chinese
> 设置权限

```
chmod u+rwx /usr/share/fonts/chinese/*
sudo mkfontscale
sudo mkfontdir
sudo fc-cache –fv
source /etc/profile
```
> 查看 fc-list
