#Mac 环境MySQL开启binlog.md
> 查看Mysql是否开启binlog

```
show variables like 'log_bin';

显示
log_bin OFF

```

需要开启 binlog

> 修改my.cnf文件支持binlog

```
smysql --help --verbose | grep my.cnf

/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf

```

因为 brew 安装的mysql /etc/my.cnf 文件

创建文件

sudo vim /etc/my.cnf

```
 #log_bin
 [mysqld]
 log-bin = mysql-bin #开启binlog
 binlog-format = ROW #选择row模式
 server_id = 1 #配置mysql replication需要定义，不能喝canal的slaveId重复

```

重启 mysql
```
 mysql.server restart

```

>在查看Mysql是否开启binlog

```
show variables like 'log_bin';

显示
log_bin ON

```
