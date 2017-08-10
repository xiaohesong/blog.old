---
title: Rails Middleware
date: 2017-08-10 11:14:39
tags:
  - rails
  - 中间件
categories:
  - 迁移
---

### Rails Middleware

- app/middleware/dalta_logger.rb

```ruby
#app/middleware/dalta_logger.rb
class DeltaLogger
  def initialize(app, formatting_char = '=', log_level = "info")
    @app = app
    @formatting_char = formatting_char
    @log_level = log_level
  end

  def call env
    request_started_on = Time.now
    @status, @headers, @response = @app.call(env)
    request_ended_on = Time.now

    Rails.logger.send(@log_level, @formatting_char * 50)
    Rails.logger.send(@log_level, "Request delta time: #{request_ended_on - request_started_on} seconds.")
    Rails.logger.send(@log_level, @formatting_char * 50)

    [@status, @headers, @response]
  end
end
```

- config/application.rb

```ruby
config.middleware.use "DeltaLogger", "*", :warn
```


##### restart server

#### [原文](http://ieftimov.com/writing-rails-middleware)

### [blog原文](http://blog.csdn.net/hesonggg/article/details/53187429)
