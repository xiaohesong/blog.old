---
title: docker+daocloud+rails部署管理
date: 2017-08-17 12:52:36
tags:
  - docker
  - 部署
  - rails
  - daocloud
categories:
  - 部署
---


#### 本文将简述如何将`docker`放在`daocloud`里管理. 将在[上一篇的基础上](http://blog.csdn.net/hesonggg/article/details/68927265)阐述.

#### 去[daocloud官网](https://www.daocloud.io/)注册登录账号之后,然后在用户中心代码托管里绑定`git`账号.

### 简单操作
1 首先创建一个项目.如`psp`, 然后选择`git` 的相关仓库.例如`psp`.(创建的项目需`git`仓库有`Dockerfile`文件.) 然后就可以创建了.

2 创建成功之后,可以找到`设置`,勾选`成功构建后设置 latest 为镜像标签`.
点击左侧的项目找到`psp`,右侧的查看详情下拉框下选择点击`手动触发`.可以选择触发的分支.我是选择了一个`docker`分支.点击确定之后便会执行.可以点击进去查看详情.一会便会构建成功.

3 执行成功之后,可以在镜像仓库里发现我们刚执行的结果.点击`部署最新版本`.填写应用名称选择主机,就可以进行下一步操作.

	- 选择主机 ,需要在对应的服务器添加监控,才可以检测到并选择.
	- 自有主机-集群管理-添加主机.可以发现有个命令,在服务器上添加就好了.

4 然后返回第3步,添加刚添加的主机.点击下一步.进去之后可以先不做配置,直接点击`立即部署`.这个时候会跳转到容器的日志部分.等待部署成功,即状态为`运行中` 这个时候我们就完成了一个简单的配置.

### 多个容器的操作
##### 上面我们完成了一个简单的容器启动.不过一个应用下来,有很多容器需要启动配置.

- 自有主机-应用
	在这里我们可以找到之前创建的一个容器.这个容器里有配置,有YAML.有日志,不过这个还只是一个单独的容器,我们需要应用其他的容易.
	注意, 容器内部的这个`yml`只是针对单个容器的配置,并不类似于`docker-compose.yml`, 我们来尝试在`daocloud`里配置`docker`的`compose`.

- 自有主机-Stack
1. `创建新Stack`,输入名称,如: `psp-compose`,这里你自定义.
2. 这里的`yml`配置值得注意,虽然`stack`的作用和`docker-compose`类似,但是语法方面还是有些细微的差别.
例如,我们直接在`docker-compose.yml`里可以对`Dockerfile`进行`build`或者指定`Dockerfile`进行`build`.但是这里就不行,这里`build`需要替换成`image`.下面是我的`stack`的例子.

```yml
version: '2'
services:
  app:
    image: daocloud.io/xiaohesong/psp_staging:latest #这个是你构建的镜像.
    command: /bin/bash start_up.sh
    privileged: false
    restart: always
    links:
    - postgres
    - redis
    ports:
    - '3000'
    environment:
    - VIRTUAL_HOST=ip #这个是nginx-proxy镜像的一个方法.为你配置nginx.
    -SECRET_KEY_BASE= xxxxxxxxx
    volumes:
    - /rails/data/log:/home/issue/www/psp/log
  web:
    image: daocloud.io/daocloud/nginx-proxy
    ports:
    - 80:80
    links:
    - app
    volumes:
    - /var/run/docker.sock:/tmp/docker.sock:ro
  postgres:
    image: postgres:9.4
    ports:
    - '5432'
    volumes:
    - /rails/main-database:/var/lib/postgresql/data
  redis:
    image: redis:3.0.5
    ports:
    - 6379:6379
    volumes:
    - /rails/cache-database:/var/lib/redis/data

```

- 这里的`nginx`镜像是`nginx-proxy`. 他提供了一个`VIRTUAL_HOST`变量,会根据指定这个变量所在的容器去配置`nginx`的配置信息.如果不用这个,自己又需要配置`nginx`的信息,例如[上一篇文章](http://blog.csdn.net/hesonggg/article/details/68927265)的`nginx`,需要指定相关的配置,去指定`Dockerfile-nginx`. 这样可能就需要你去创建一个镜像.类似`psp`,自己构建一个镜像到`daocloud`,然后再指定构造的镜像.

-
创建`Stack`的时候,把这个写在`yml`文件里(因项目差异,这里有些需要你自己去更改.),然后选择主机创建部署,会跳转到日志,等待完成.
完成之后会发现,在`自有主机-应用`里创建了我们在`Stack`里加的相关容器.

`docker`和`daocloud`的大体流程类似这样,具体的细节还是需要去完善.

#### 番外: 比较坑的地方

- 服务器
之前阿里活动,9块钱半年的服务器搞了一台.发现配置好了站点之后,`ip`访问不了. 服务器上访问是ok的.

解决方案:
进入阿里云控制台 --> 云服务器 ECS --> 网络和安全 --> 安全组 --> 配置规则 --> 快速创建规则
在里面选择`HTTP(80)`, `授权对象`不清楚的就直接`0.0.0.0/0`
