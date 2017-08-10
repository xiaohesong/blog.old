---
title: 使用sinatra实现简单的crud功能
date: 2016-11-16 15:53
tags:
  - sinatra
categories:
  - 迁移
---


### Sinatra

```ruby
gem install sinatra
```

-  hello_sinatra

```ruby
mkdir -p hello_sinatra
cd hello_sinatra
```

vim hello_sinatra.rb
```ruby
require "sinatra"
get "/" do
  "Hello, world!"
end
```
 然后
##### ruby hello_sinatra.rb

localhost:4567

修改hello_sinatra.rb
```ruby
require "sinatra"
get "/post/:id" do
  "Hello, world!参数是#{params}"
end
```

然后刷新页面，会发现，有报错，不生效。
> Note that every time we change the Ruby file, we’ll need to restart the server

这样每次修改一下，不是很讨厌吗？？
shotgun可以避免这个问题。
```ruby
gem install shotgun
```
然后启动的时候注意一下。
```ruby
ruby hello_sinatra.rb -> shotgun hello_sinatra.rb
```
##### 这个时候的端口，由4567改变成了9393

[Creating a Basic Web App With Ruby and Sinatra](http://blog.roberteshleman.com/2014/06/26/creating-a-basic-web-app-with-ruby-and-sinatra/)

- build sinatra app

```shell
mkdir pubic-bookmarks-sinatra
cd pubic-bookmarks-sinatra
```
###### vim public-bookmarks.rb
```ruby
require 'sinatra'
get '/' do
 "#{['Hello', 'Hi', 'Hey', 'Yo'][rand(4)]} World!"
end
```

###### vim Gemfile
```ruby
source 'https://rubygems.org' #gem源自行修改
gem 'sinatra'
ruby '2.1.2'
```
> Note: bundle

##### vim config.ru
```ruby
require './public-bookmarks'
run Sinatra::Application
```
```ruby
rackup -p 4567
```

- layout

```ruby
# public-bookmarks.rb
require 'sinatra'
get '/' do
  erb :hello_world
end
```
```ruby
mkdir -p views
```

```html
#vim views/hello_world.erb
<%= ['Hello', 'Hi', 'Hey', 'Yo'][rand(4)] %> World!
```

###### layout
```html
<!DOCTYPE html>
<html>
<head><title>PublicBookmarksSinatra</title></head>
<body>

<%= yield %>

</body>
</html>
```

- Use ActiveRecord

```ruby
# vim Gemfile
gem 'sinatra-activerecord'
gem 'pg'
gem 'rake'
```

> Note: bundle

- config/database.yml

```ruby
#vim database.yml
development:
  adapter: postgresql
  database: public-bookmarks-sinatra_development
  host: localhost
  username: postgres
  password: root
```

```ruby
# vim Rakefile
require 'sinatra/activerecord/rake'
require './public-bookmarks'
```

##### 然后在主文件里引入
```ruby
# public-bookmarks.rb
require 'sinatra'
require 'sinatra/activerecord'
get '/' do
  db_time = database.connection.execute('SELECT CURRENT_TIMESTAMP').first['now']
  request.logger.info "DB time is #{db_time}"
  erb :hello_world
end
```

##### 现在重新启动下试试
```ruby
rake db:create
rackup -p 4567
```
##### 可以在后台日志看见相关的输出信息。

- Build model

```ruby
# 首先指定并创建一个migration的名字为create_public_bookmarks
rake db:create_migration NAME=create_public_bookmarks
```

##### 然后打开这个文件 并且写入相关字段
```ruby
# db/migrate/YYYYMMDDHHMMSS_create_public_bookmarks.rb
class CreatePublicBookmarks < ActiveRecord::Migration
  def change
    create_table :public_bookmarks do |t|
      t.string :title
      t.string :url
      t.text :description
      t.string :submitter_email

      t.timestamps
    end
    add_index :public_bookmarks, :url, unique: true
  end
end
```

##### 执行迁移文件
```ruby
rake db:migrate
```

##### 接下来创建 public_bookmark model
```shell
mkdir models
```

```ruby
# vim models/public_bookmark.rb
class PublicBookmark < ActiveRecord::Base
end
```

##### 接下来我们需要在主文件里指定model
```ruby
#public-bookmarks.rb
...
require './models/public_bookmark'
...
```

- Build Controllers

> Note: 在一些时候，我们需要类似rails的notice等闪存方式来提示信息。所以需要安装一个gem.

```ruby
# Gemfile
...
gem 'rack-flash3'
...
```

> Note: bundle

##### 这个和model类似，需要在主文件内部指定实用。

```ruby
# public-bookmarks.rb
...
require './models/public_bookmark'
require 'rack-flash'

enable :sessions
use Rack::Flash
...
```

##### 基本的配置差不多就这样，接下来，可以给controller填充一些action
```ruby
get '/public_bookmarks' do
  @public_bookmarks = PublicBookmark.all
  erb :'public_bookmarks/index'
end

get '/public_bookmarks/new' do
  @public_bookmark = PublicBookmark.new
  erb :'public_bookmarks/new'
end

get '/public_bookmarks/:id' do
  @public_bookmark = PublicBookmark.find(params[:id])
  erb :'public_bookmarks/show'
end

post '/public_bookmarks/create' do
  @public_bookmark = PublicBookmark.new(params[:public_bookmark])
  if @public_bookmark.save
    flash[:notice] = 'Public bookmark successfully created!'
    redirect to("public_bookmarks/#{@public_bookmark.id}")
  else
    erb :'public_bookmarks/new'
  end
end
```
##### 然后就可以在views下创建一个public_bookmarks文件夹，创建对应的view.
``` html
#views/public_bookmarks/index.erb:

<span>Listing public_bookmarks</span>

<p id="notice"><%= flash[:notice] %></p>

<table>
  <thead>
    <tr>
      <th>Title</th>
      <th>Url</th>
      <th>Description</th>
      <th>Submitter email</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @public_bookmarks.each do |public_bookmark| %>
      <tr>
        <td><%= public_bookmark.title %></td>
        <td><%= public_bookmark.url %></td>
        <td><%= public_bookmark.description %></td>
        <td><%= public_bookmark.submitter_email %></td>
        <td>
          <a href="/public_bookmarks/<%= public_bookmark.id %>">
            Show
          </a>
        </td>
        <% if @authenticated %>
          <td>
            <form action="/public_bookmarks/destroy/<%= public_bookmark.id %>" method='post'>
              <input type='submit' value='Destroy' onclick="return confirm('Are you sure?')">
            </form>
          </td>
        <% end %>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<a href='/public_bookmarks/new'>
  New Public Bookmark
</a>


#views/public_bookmarks/show.erb:

<p id="notice"><%= flash[:notice] %></p>

<p>
  <strong>Title:</strong>
  <%= @public_bookmark.title %>
</p>

<p>
  <strong>Url:</strong>
  <%= @public_bookmark.url %>
</p>

<p>
  <strong>Description:</strong>
  <%= @public_bookmark.description %>
</p>

<p>
  <strong>Submitter email:</strong>
  <%= @public_bookmark.submitter_email %>
</p>

<a href='/public_bookmarks'>
  Back
</a>


# views/public_bookmarks/new.erb:

<span>New public_bookmark</span>

<%= erb :'public_bookmarks/form' %>

<a href='/public_bookmarks'>
  Back
</a>



# views/public_bookmarks/form.erb:

<form action="/public_bookmarks/create" method='post'>
  <% if @public_bookmark.errors.any? %>
    <div id="error_explanation">
      <span><%= pluralize(@public_bookmark.errors.count, "error") %> prohibited this public_bookmark from being saved:</span>

      <ul>
      <% @public_bookmark.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <label for="public_bookmark_title">Title</label><br>
    <input id="public_bookmark_title" name="public_bookmark[title]" type="text">
  </div>
  <div class="field">
    <label for="public_bookmark_url">Url</label><br>
    <input id="public_bookmark_url" name="public_bookmark[url]" type="text">
  </div>
  <div class="field">
    <label for="public_bookmark_description">Description</label><br>
    <textarea id="public_bookmark_description" name="public_bookmark[description]"></textarea>
  </div>
  <div class="field">
    <label for="public_bookmark_submitter_email">Submitter email</label><br>
    <input id="public_bookmark_submitter_email" name="public_bookmark[submitter_email]" type="text">
  </div>
  <div class="actions">
    <input type="submit" value="Create Public bookmark">
  </div>
</form>
```

### [原文](https://www.airpair.com/ruby-on-rails/posts/rails-vs-sinatra)

### [github](https://github.com/xiaohesong/public-bookmarks-sinatra)

### [blog原文](http://blog.csdn.net/hesonggg/article/details/53187411)
