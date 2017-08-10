---
title: 阿里云ECS服务器安装postgresql无法启动的问题
date: 2016-12-14 12:05
tags:
  - pg
  - 运维
categories:
  - 迁移
---
安装很简单：

```shell
sudo apt-get install postgresql #安装这个库
```
```shell
sudo su postgres -c psql postgresql #这里是进入数据库下
```

然后报错进不去。locale 语言的设置问题。如下:

```shell
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
LANGUAGE = "unset",
LC_ALL = "unset",
LC_CTYPE = "UTF-8",
LANG = "en_US.UTF-8"
are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
...
...
```

这里经过排查，可以知道是因为系统语言的问题。
解决方式：

```shell
sudo locale-gen en_US en_US.UTF-8
sudo dpkg-reconfigure locales
```
然后执行第二部尝试是否成功，如果不成功，会发现对应的
LC_ALL以及LC_LANGUAGE还是空的（unset）,可以这样解决：
```shell
export LANGUAGE="en_US.UTF-8"
echo 'LANGUAGE="en_US.UTF-8"' >> /etc/default/locale
echo 'LC_ALL="en_US.UTF-8"' >> /etc/default/locale
```
设置好之后，重新登陆服务器。
再次尝试执行 ```  sudo su postgres -c psql postgresql ``` 尝试登入数据库。
希望你能成功。很可惜，我这里是失败了。
报错：
```shell
psql: could not connect to server: No such file or directory

Is the server running locally and accepting connections on Unix domain socket"/var/pgsql_socket/.s.PGSQL.5432"?
```
然后尝试重启postgresql服务
```shell
sudo service postgresql restart
```
会有一个报错：
```shell
No PostgreSQL clusters exist; see “man pg_createcluster”
```
解决方式这般：
```shell
sudo pg_createcluster 9.3 main --start
```

然后进入数据库尝试。是可以进入的。
进入之后，我们要改变数据库的密码

```shell
ALTER USER postgres WITH PASSWORD 'xiaozhu';  #修改你的密码
\q
sudo passwd postgres #修改postgres用户的密码
```


#### 这些是我在服务器上第一次碰到这样的问题，希望能对你有所帮助。

#### [blog原文](http://blog.csdn.net/hesonggg/article/details/53637073)
