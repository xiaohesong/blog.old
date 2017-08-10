---
title: ubuntu查看postgresql操作日志
date: 2017-08-10 10:21:24
tags:
  - 运维
  - 日志
categories:
  - 迁移
---

#### 配置
- pg(9.3)
`/etc/postgresql/9.3/main`下找到`postgresql.conf`

```bash
# sudo vim postgresql.conf
logging_collector = on # 默认是off,记得去除注释
log_directory = 'pg_log' # 取消注释
log_statement = 'none'	# none, ddl, mod, all, 这里设置成aoo
```
注意: `pg_log`在`$pgdata`文件夹里.
查找`$pgdata`
```shell
ps auxw |  grep postgres | grep -- -D
```
可以找到对应的文件夹位置.我的是在`/var/lib/postgresql/9.5/main`下面.

目前是没有这个`pg_log`文件夹,因为我们还没有重启生效这个操作.

```shell
sudo service postgresql restart
```

然后运行了`pg`之后,可以发现有`pg_log`
```shell
tail f /var/lib/postgresql/9.5/main/postgresql-2016-10-24_175049.log
```
