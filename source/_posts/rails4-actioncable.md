---
title: rails4 actioncable
date: 2017-05-26 13:13
tags:
  - rails
categories:
  - 迁移
---


15年的时候,出了rails5,有个新特性actioncable.刚好公司有个关于推送的需求,但是Rails的版本是Rails4.想使用`actioncable`. 找了很久找到了一个可以使用的官方的gem. 16年的时候在公司项目中用了进去.
现在打算把这个写出来,因为好像很多人都不知道rails4有这个gem可以使用.

### [actioncable](https://github.com/rails/actioncable/tree/archive)

在`gemfile`添加
```ruby
gem 'actioncable', github: 'rails/actioncable', branch: 'archive'
```
ps: 为了防止作者把分支删除,可以先备份一个到自己的仓库里.

`bundle`之后需要配置一番,这个可不像rails5那么棒,不用配置都行.
`application.js`
```js
...
//= require cable
...
```

在`app` 文件夹下创建相关的目录
`channels/application_cable/channel.rb`
```ruby
module ApplicationCable
  class Channel < ActionCable::Channel::Base
  end
end
```
`channels/application_cable/connection.rb`
```ruby
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :push_store

    def connect
      self.push_store = find_verified_user.store
    end

    protected

    def find_verified_user
      session_key = Rails.application.config.session_options[:key]
      user_id = cookies.encrypted[session_key]['user_id']

      if verified_user = User.find_by(id: user_id)
        verified_user
      else
        reject_unauthorized_connection
      end

    end
  end
end
# note: 因为websocket不支持session传输，所以此处需要由cookies解密
```

`channels/orders_channel.rb`
```ruby
class OrdersChannel < ApplicationCable::Channel
  def subscribed
    stream_from "order_channel_#{push_store.id}"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
end
# subscribed是连接过来发生的行为，unsubscribed是断开连接发生的行为。
```

我们在上面将最基本的桥梁搭建好了，那么我得去创建一个消费者去使用他。 在 `app/javascripts/`下创建一个 `app.es6`
```js
let App = {};

App.cable = Cable.createConsumer(`/cable`);

App.messaging = App.cable.subscriptions.create('OrdersChannel', {
  received: function(data) {
    $(this).trigger('received', data);
  }
});

```
消费者创建好之后，我们需要播报一条消息，让他去监听行为并做出处理 在 `app/jobs`下创建 `order_broadcast_job.rb`
```ruby
class OrderBroadcastJob < ActiveJob::Base
  queue_as :default

  def perform(store)
    ActionCable.server.broadcast "order_channel_#{store.id}", { message: "message", user: "current_user" }
  end
end
```
可以发现，这个任务就是去向order_channel播报一条通知，可以理解为：‘我发布了一个广播，指定进入到`order_channel`. 连接上来，自然而然的触发了`subscribed`。广播指定的通道，会去匹配`stream_from`.

上面我们发布了一个广告有任务了，那么消费者就会积极相应，主动接收任务。 接收了任务之后，就会发生对应的行为。`app/javascripts/order.es6`
```js
$(function($) {
  $(App.messaging).on('received', function(event, data) {
    let count = 0
    const message = () => {
      count++;
      if(count%2==1){
        document.title='【你有新的消息】'
      }else {
        document.title='【　　　　　　】'
      }
    }
    setInterval(message, 800)

  });

});
```

新建cable文件夹，与app文件夹同级 `cable/config.ru`
```ruby
require ::File.expand_path('../../config/environment',  __FILE__)
Rails.application.eager_load!

require 'action_cable/process/logging'

run ActionCable.server
```
在`bin`文件夹下创建`cable`脚本
```ruby
#!/bin/bash
bundle exec puma -p 28080  cable/config.ru
```
本地使用的时候,直接 `$bin/cable`就好.

关于部署可以参考我的笔记.[actioncable发布](https://ruby-china.org/notes/3598)

#### [原文](http://blog.csdn.net/hesonggg/article/details/72765606)
