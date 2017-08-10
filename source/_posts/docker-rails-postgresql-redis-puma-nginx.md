---
title: docker+rails+postgresql+redis+puma+nginx
date: 2017-03-31 17:10
tags:
  - docker
  - 运维
  - 部署
categories:
  - 迁移
---

看了两天的docker相关文章，就尝试把一个现有的小blog换成docker部署尝试一番．
前期有个疑问，capistrano一类的工具也可以很方便的部署，为什么还要docker．
有人说保持环境一致，这个不多做评价，不过个人感觉，rails分为三个环境，各有优势，为何要保持一致．
在我看来，capistrano和docker的主要区别，还是一个适合多机部署，一个适合单机部署．也有一些人说docker更适合微架构，不过我觉得多机部署也没有什么区别把，单点故障之类的都可以．
有些扯远了，还是言归正传吧．

我们首先得安装docker及相应的工具．

- 安装docker

	```shell
	curl -sSL https://get.daocloud.io/docker | sh
	```
	这里可以把docker添加到组里,安装成功之后会有这么一句话．
	`
	sudo usermod -aG docker name
	`

- 安装docker-compose

	`
	curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-Linux-x86_64 > \
  /tmp/docker-compose && \
  chmod +x /tmp/docker-compose &&
  sudo mv /tmp/docker-compose /usr/local/bin
  `
##### 然后我们验证是否安装成功．

`
	docker --version
	docker-compose --version
`

这里主要是针对现存的项目进行处理．假设项目名称是 `psp`.
我们在项目主目录下新建一个文件
`Dockerfile`
```shell
FROM ruby:2.3.1　＃基本镜像

#author info
MAINTAINER Xiaozhu didmehh@163.com

#基本的依赖
RUN apt-get update && apt-get install -y build-essential libpq-dev nodejs

＃设置环境变量
ENV RAILS_ENV production
ENV RACK_ENV production
上面两个是指定应用环境
ENV RAILS_ROOT /home/issue/www/psp
＃创建工作目录
RUN mkdir -p $RAILS_ROOT

＃指定工作目录
WORKDIR $RAILS_ROOT

#处理gem
COPY Gemfile Gemfile
COPY Gemfile.lock Gemfile.lock
RUN gem install bundler
RUN bundle install

# 复制puma的配置
COPY config/puma.rb config/puma.rb

# 复制主目录
COPY . .
＃　暴露3000
EXPOSE 3000

＃运行shell脚本
CMD bash start_up.sh
```
#### 上面的是Dockerfile的基本配置．发现上面有出现 `config/puma.rb`	以及　`start_up.sh`.
#### `config/puma.rb`在rails5里会自动创建，可以自己找．
#### `start_up.sh` 内容如下：

```shell
#!/bin/bash
#　处理静态资源
bundle exec rake assets:precompile

#启动puma
bundle exec puma -C config/puma.rb
```

这个是基本的配置，针对rails服务．下面来看看nginx的处理．
##### 在主目录下创建 `Dockerfile-nginx`

```shell
FROM nginx

RUN apt-get update -qq && apt-get -y install apache2-utils

ENV RAILS_ROOT /home/issue/www/psp

WORKDIR $RAILS_ROOT

RUN mkdir log

COPY public public/

COPY config/nginx.conf /tmp/docker_example.nginx
RUN envsubst '$RAILS_ROOT' < /tmp/docker_example.nginx > /etc/nginx/conf.d/default.conf
# RUN rm -rf /etc/nginx/sites-available/default
# ADD config/nginx.conf /etc/nginx/sites-enabled/nginx.conf

EXPOSE 80

CMD [ "nginx", "-g", "daemon off;" ]

```
到这里，我们把基本的Dockerfile给创建了．
接下来我们需要在项目里写个nginx配置文件
```shell
	#config/nginx.conf

	upstream puma {
	  server app:3000;
	}


	server {

	  listen 80;

	  client_max_body_size 4G;
	  keepalive_timeout 10;

	  error_page 500 502 504 /500.html;
	  error_page 503 @503;

	  server_name xiaohesong.com;
	  root /home/issue/www/psp/public;
	  try_files $uri/index.html $uri @puma;

	  access_log /home/issue/www/psp/log/nginx.access.log;
	  error_log /home/issue/www/psp/log/nginx.error.log;

	  location @puma {
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header Host $http_host;
	    proxy_redirect off;

	    proxy_pass http://puma;
	    # limit_req zone=one;
	  }

	  location ^~ /assets/ {
	    gzip_static on;
	    expires max;
	    add_header Cache-Control public;
	  }

	  location = /50x.html {
	    root html;
	  }

	  location = /404.html {
	    root html;
	  }

	  location @503 {
	    error_page 405 = /system/maintenance.html;
	    if (-f $document_root/system/maintenance.html) {
	      rewrite ^(.*)$ /system/maintenance.html break;
	    }
	    rewrite ^(.*)$ /503.html break;
	  }

	  if ($request_method !~ ^(GET|HEAD|PUT|PATCH|POST|DELETE|OPTIONS)$ ){
	    return 405;
	  }

	  if (-f $document_root/system/maintenance.html) {
	    return 503;
	  }

	  location ~ \.(php|html)$ {
	    return 405;
	  }
	}

```

####上面配置完了nginx以及主应用，那么我们得把他们关联起来．
我们需要`docker-compose`
在主目录下创建`docker-compose.yml`

```shell
version: '2'
services:
  app:
    build: .
    command: bundle exec puma -C config/puma.rb
    volumes:
      - .:/home/issue/www/psp
    expose:
      - "3000"
    links:
      - postgres
      - redis
    env_file:
      - .secret.env
  web:
    build:
      context: .
      dockerfile: Dockerfile-nginx
    links:
      - app
    ports:
      - "80:80"
  postgres:
    image: postgres:9.4
    ports:
      - "5432"
    volumes:
      - ~/main-database:/var/lib/postgresql/data
  redis:
    image: redis:3.0.5
    ports:
      - '6379:6379'
    volumes:
      - ~/cache-database:/var/lib/redis/data
```

- 可以发现在redis以及postgres加上了volumes.这个是可以设置共享数据，挂在数据卷，便于持久化存储数据．
- redis以及postgres都是直接从远程仓库拷贝下来的镜像．[hub docker](https://hub.docker.com/explore/)
- volumes前面的宿主机可以通过 `docker volume create --name cache-database`　来创建,如果不创建，会默认生成．可以通过 `docker volume ls` 查看.
设置的别名如果过多，一个个的创建很麻烦，你也可以这样处理．
```shell
version: '2'
services:
  app:
    ...
  ...
    ...
volumes:
  cache-database:
  main-database:
```

- links是把需要的服务拿过来使用．例如app作为主服务，需要持久化存储的数据库．
- 配置数据库的用户密码．
```shell
postgres:
    image: postgres:9.4
	...
	environment:
	  POSTGRES_USER: pg_user
      POSTGRES_PASSWORD: secret password
```

- app的配置里，我们有个env_file,这个可以在主目录下创建一个.env的文件．
可以用来存放一些环境变量，或者可以直接shell脚本直接写文件到电脑系统配置里
我们这里存放的主要是secret：
```shell
SECRET_KEY_BASE=ecd694a53572357f98c4644991ef5d6e27d46a1dc18d605b7e58c7143ff0122c868b63f771788c82fe825fd6945dfa0f322ec73dddcf19d1e6c85234a66eae44
```
SECRET_KEY_BASE可以通过 `RAILS_ENV=production bundle exec rake secret`　生成.

#### 接下来配置数据库．

```ruby
#config/database.yml

default: &default
  adapter: postgresql
  encoding: unicode
  pool: 5
  timeout: 5000
  username: postgres
  # please see the update below about using hostnames to
  # access linked services via docker-compose
  host: postgres #是 docker-compose.yml中的key,与app,web同列，redis同样
  port: 5432
  password: #<%= ENV['POSTGRES_PASSWORD'] %>


development:
  <<: *default
  database: psp_dev

test:
  <<: *default
  database: psp_test

production:
  <<: *default
  database: psp_pro
```

```ruby
# config/redis.yml
defaults: &defaults
  host: redis
  port: 6379

development:
  <<: *defaults

test:
  <<: *defaults

staging:
  <<: *defaults

production:
  <<: *defaults
```

接下来，我们构建这些个服务．

```shell
docker-compose build
```
这里可能需要一些时间，构建好之后，我们先启动测试一下，可以不要以守护状态运行，直接:
```shell
docker-compose up
```

不出意外．可以发现启动成功．我们看下启动了哪些容器．打开一个标签页，运行以下命令查看:

```shell
docker ps
```
会有以下的输出：
```shell
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
f716c3ee2414        psp_web             "nginx -g 'daemon ..."   About an hour ago   Up About an hour    0.0.0.0:80->80/tcp, 443/tcp   psp_web_1
57702197c38c        psp_app             "bundle exec puma ..."   About an hour ago   Up About an hour    3000/tcp                      psp_app_1
415edf596c26        redis:3.0.5         "/entrypoint.sh re..."   About an hour ago   Up About an hour    0.0.0.0:6379->6379/tcp        psp_redis_1
cecf281102b3        postgres:9.4        "docker-entrypoint..."   About an hour ago   Up About an hour    0.0.0.0:32770->5432/tcp       psp_postgres_1
```
启动了也存在问题呀，数据库没有，那我们就创建一个数据库．
```shell
docker-compose run app rake db:create
docker-compose run app rake db:migrate
```

如果想删除所有的容器，可以运行下面这个命令:
```shell
docker rm `docker ps --no-trunc -aq`
```

## Ps: [源码](https://github.com/xiaohesong/psp/tree/docker)

###参考
- shell脚本推荐[鸟哥linux私房菜](http://linux.vbird.org/)
- [docker基本知识参考随风的系列博客](https://www.rails365.net/articles/an-zhuang-docker-yi)
- [RAILS 5 AND DOCKER (PUMA, NGINX)](http://codepany.com/blog/rails-5-and-docker-puma-nginx/)
- [connect database](https://docs.docker.com/compose/rails/#connect-the-database)

###外传，遇到的问题:

- 问题一：无法build容器.
执行`docker-compose build`　无法正常构建．
![如下图所示:](http://img.blog.csdn.net/20170401113256338?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGVzb25nR0c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
折腾了很久，换源也不行，[在google找到了一个类似的](https://github.com/docker-library/rails/issues/43) .把代理关了就好．．．．

- 问题二：nginx只有　` welcome to nginx`
没有正常启动项目，就是没有和项目绑定在一起．
找了很久，最后经过各种排查以及尝试．把nginx的`Dockerfile-nginx` 修改一下：
```shell
COPY config/nginx.conf /tmp/docker_example.nginx
RUN envsubst '$RAILS_ROOT' < /tmp/docker_example.nginx > /etc/nginx/conf.d/default.conf
# RUN rm -rf /etc/nginx/sites-available/default
# ADD config/nginx.conf /etc/nginx/sites-enabled/nginx.conf
```
可以发现下面的两行被注释了，以前习惯的把配置文件放在`/etc/nginx/sites-enabled/nginx.conf` 下面．这次就不行了．
这是为什么？进入nginx容器看下.
先找到nginx的容器id, 通过`docker ps` 查看当前运行的容器．假设输出如下：
```
CONTAINER ID     |   IMAGE       |        COMMAND         |         ...                   
f716c3ee2414	 |		...		 |			...			  |			...					 
```
然后我们进入这个容器: `docker exec -it f716c3ee2414 bash`
进入容器之后 `cat /etc/nginx/nginx.conf　| grep include` 会发现输出一下内容：
`
	include       /etc/nginx/mime.types;
    include /etc/nginx/conf.d/*.conf;
`
这个　`include /etc/nginx/conf.d/*.conf;`　是包含配置文件的路劲．都在这个下面，然而我们改的是在`/etc/nginx/sites-enabled/*` 下面．所以说，配置文件没有生效，使用的默认的default配置．

- 问题三: 启动时出错，`docker-compose up` 出错．
出错是:
```shell
ERROR: for redis  driver failed programming external connectivity on endpoint psp_redis_1 (13693b22524bf940932c25f9b7f5dac45671f68fbc04fad1a76ed08274db5699): Error starting userland proxy: listen tcp 0.0.0.0:6379: bind: address already in use
Traceback (most recent call last):
  File "<string>", line 3, in <module>
  File "compose/cli/main.py", line 63, in main
AttributeError: 'ProjectError' object has no attribute 'msg'
docker-compose returned -1
```
显示已经存在，很怪异吧．经过排查，是因为本地的redis已经启动．关了本地的redis服务就好．
` sudo service redis-server stop`
然后重启，`docker-compose up`　发现不会redis出错地址绑定关系．
又出现一个错误．
```shell
ERROR: Named volume "db:/var/lib/postgresql/data:rw" is used in service "db" but no declaration was found in the volumes section.
```
这个是因为语法错误，如下：
```shell
redis:
   ...
   volumes:
     - cache-redis:/var/lib/redis/data
```
这样就会出错，换一种写法,如下：

```shell
volumes:
  - ~/cache-redis:/var/lib/redis/data
```
### Note: 目前对docker知之甚少，后期会补充，有不是的地方，恳请留言，谢谢! 后期会继续补充!



### [blog原文](http://blog.csdn.net/hesonggg/article/details/68927265)
