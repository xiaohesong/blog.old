---
title: rails capistrano nginx puma部署
date: 2017-08-10 09:44:57
tags:
  - 运维
  - 部署
categories:
  - 迁移
---

项目进入测试的阶段，要部署到staging环境进行监测。之前一直是使用的passanger服务器来跑的。最近换了puma，来说下总结吧。
具体的环境安装就不再阐述。
### 相关库的添加
- 添加`gem`
首先需要在gemfile.rb添加:
``` ruby
group :development do
  gem 'capistrano', '~> 3.3.0'
  gem 'capistrano-rvm'
  gem 'capistrano-rails-console'
  gem 'capistrano-bundler', '~> 1.1.2'
  gem 'capistrano-rails', '~> 1.1'
  gem 'capistrano3-puma',   require: false
  gem 'capistrano-sidekiq'
end
```

- `bundle`

- `cap install`.
会有如下文件生成
```shell
mkdir -p config/deploy
create config/deploy.rb
create config/deploy/staging.rb
create config/deploy/production.rb
mkdir -p lib/capistrano/tasks
create Capfile
Capified
```

### 相关配置
- `Capfile`

```ruby
# Load DSL and set up stages
require 'capistrano/setup'

# Include default deployment tasks
require 'capistrano/deploy'

# Include tasks from other gems included in your Gemfile
#
# For documentation on these, see for example:
#
#   https://github.com/capistrano/rvm
#   https://github.com/capistrano/rbenv
#   https://github.com/capistrano/chruby
#   https://github.com/capistrano/bundler
#   https://github.com/capistrano/rails
#   https://github.com/capistrano/passenger
#
require 'capistrano/rvm'
# require 'capistrano/rbenv'
# require 'capistrano/chruby'
require 'capistrano/bundler'
require 'capistrano/rails/assets'
require 'capistrano/rails/migrations'
# require 'capistrano/passenger'
require 'capistrano/puma'
require 'capistrano/sidekiq'

# Load custom tasks from `lib/capistrano/tasks' if you have any defined
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
```

无非就是添加了几个require．（`gemfile`添加的`capistrano`相关`gem`可以`require`）

- `config/deploy.rb`

```ruby
	# config valid only for current version of Capistrano
lock '3.3.5'

set :application, 'appname'
set :repo_url, 'git@github.com:example/appname.git'
# Default branch is :master
# ask :branch, proc { `git rev-parse --abbrev-ref HEAD`.chomp }.call

# Default deploy_to directory is /var/www/my_app_name
set :deploy_to, '/home/ubuntu/www/appname'
set :html_deploy_to, "#{fetch(:deploy_to)}/html"
# Default value for :scm is :git
set :scm, :git
set :user,            'ubuntu'
# Default value for :format is :pretty
# set :format, :pretty

# Default value for :log_level is :debug
# set :log_level, :debug

# Default value for :pty is false
# set :puma_init_active_record, true  # Change to false when not using ActiveRecord
# Default value for :linked_files is []
set :linked_files, fetch(:linked_files, []).push('config/database.yml', '.ruby-version', '.ruby-gemset')
# Default value for linked_dirs is []
set :linked_dirs, fetch(:linked_dirs, []).push('bin', 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system')

# Default value for default_env is {}
# set :default_env, { path: "/opt/ruby/bin:$PATH" }

# Default value for keep_releases is 5
set :keep_releases, 5
set :default_env, { rvm_bin_path: '~/.rvm/bin' }

# namespace :puma do
#   desc 'Create Directories for Puma Pids and Socket'
#   task :make_dirs do
#     on roles(:app) do
#       execute "mkdir #{shared_path}/tmp/sockets -p"
#       execute "mkdir #{shared_path}/tmp/pids -p"
#     end
#   end
#
#   before :start, :make_dirs
# end


set :puma_threads,    [4, 16]
set :puma_workers,    0

# Don't change these unless you know what you're doing
set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/www/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/appname-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.error.log"
set :puma_error_log,  "#{release_path}/log/puma.access.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w(~/.ssh/id_rsa.pub) }
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true  # Change to false when not using ActiveRecord

## Defaults:
# set :scm,           :git
set :branch,        :staging
# set :format,        :pretty
# set :log_level,     :debug
# set :keep_releases, 5

namespace :puma do
  desc 'Create Directories for Puma Pids and Socket'
  task :make_dirs do
    on roles(:app) do
      execute "mkdir #{shared_path}/tmp/sockets -p"
      execute "mkdir #{shared_path}/tmp/pids -p"
    end
  end

  before :start, :make_dirs
end

namespace :deploy do
  desc "Make sure local git is in sync with remote."
  task :check_revision do
    on roles(:app) do
      unless `git rev-parse HEAD` == `git rev-parse origin/master`
        puts "WARNING: HEAD is not the same as origin/master"
        puts "Run `git push` to sync changes."
        exit
      end
    end
  end

  desc 'Initial Deploy'
  task :initial do
    on roles(:app) do
      before 'deploy:restart', 'puma:start'
      invoke 'deploy'
    end
  end

  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      invoke 'puma:restart'
    end
  end

  before :starting,     :check_revision
  after  :finishing,    :compile_assets
  after  :finishing,    :cleanup
  after  :finishing,    :restart
end

# ps aux | grep puma    # Get puma pid
# kill -s SIGUSR2 pid   # Restart puma
# kill -s SIGTERM pid   # Stop puma

```

- config/deploy/staging.rb

```ruby
role :app, %w{ubuntu@114.11.11.11}
role :web, %w{ubuntu@114.11.11.11}
role :db,  %w{ubuntu@114.11.11.11}
# 183.156.126.167
set :deploy_to, "/home/ubuntu/www/appname"
# Extended Server Syntax
# ======================
# This can be used to drop a more detailed server definition into the
# server list. The second argument is a, or duck-types, Hash and is
# used to set extended properties on the server.
# set :bundle_gemfile,  "ajax/Gemfile"

# set :rvm_type, :ubuntu
# server '183.156.126.167', user: 'deploy', roles: %w{web app db}, my_property: :my_value
set :branch, 'staging'

set :rails_env, "staging"

set :nginx_server_name, "example.com"

set :rvm_ruby_version, '2.3.0'
# set :rvm_type, :oss
set :monit_role, :all

set :html_branch, 'development'
```


- `vim config/nginx.conf`

```ruby
upstream puma {
  server unix:///home/ubuntu/www/app/shared/tmp/sockets/appname-puma.sock;
}

server {
  listen 80;
  server_name example.com;

  root /home/ubuntu/www/app/current/public;
  access_log /home/ubuntu/www/app/current/log/nginx.access.log;
  error_log /home/ubuntu/www/app/current/log/nginx.error.log info;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @puma;
  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}
```

上面的三个文件需要根据自己的实际情况去修改．
然后我们需要登录到远程服务器去再操作.
```bash
sudo rm /etc/nginx/sites-enabled/default	#删除默认的nginx配置
sudo ln -nfs "/home/ubuntu/www/appname/current/config/nginx.conf" "/etc/nginx/sites-enabled/appname"
# 软连接项目nginx配置到nginx服务器
```

ok,重启`nginx`.
然后可以线上查看下puma进程
`ps aux| grep puma`

如果页面是空白，请重启`puma`尝试。
`cat puma.pid | xargs kill -SIGUSR2`


#### 注意
- Note: If you make changes to your config/nginx.conf file, you'll have to reload or restart your Nginx service on the server after deploying your app:
sudo service nginx restart

### 遇到的问题
- 部署完成之后，部署的日志显示puma重启成功，但是实际是未启动成功.

检查了很久，看了日志报错

`bundler: failed to load command: puma (/home/ubuntu/www/appname/shared/bundle/ruby/2.3.0/bin/puma)`

原因是因为deploy.rb中的application 名字和服务器路径的appname不一致导致。改下名字就好了。

- puma.sock无法改变名字

先在`deploy.rb`中改变名字　如:
```ruby
"unix://#{shared_path}/tmp/sockets/new-puma.sock"
```
然后在`nginx.conf`中改变:
```ruby
"unix://#{shared_path}/tmp/sockets/new-puma.sock"
```
提交到远程分支之后，初始化`puma`的配置,如下：
```ruby
bundle exec cap staging puma:config
```
这个时候，他会在`/home/ubuntu/www/appname/shared/`下新建`puma.rb`.
因为改变了`nginx.conf`，所以需要重新加载`nginx`:
```shell
sudo service nginx reload
```
重启`nginx`
```shell
sudo service nginx restart
```
这个时候，你会发现`/home/ubuntu/www/appname/shared/tmp/sockets` 存在一个`new-puma.sock`
