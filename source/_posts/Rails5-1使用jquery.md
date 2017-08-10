---
title: ' Rails5.1使用jquery'
date: 2017-06-12 11:21
tags:
  - rails
categories:
  - 迁移
---

`rails5.1`的变化还真是大,对于`js`的拥抱很友好.
[Rails5.1变化](http://weblog.rubyonrails.org/2017/2/23/Rails-5-1-beta1/)

1. 前端的管理工具,类似`bundler`.

```ruby
rake yarn:install
```
ps : [安装node](https://github.com/creationix/nvm#install-script)
ps : [安装yarn](https://yarnpkg.com/lang/en/docs/install/#linux-tab)

2. 利用`yarn`添加`juqery`

```ruby
  yarn add jquery
```

3. `application.js`如下
```ruby
//= require jquery
//= require jquery_ujs
...
```


#### [blog原文](http://blog.csdn.net/hesonggg/article/details/73089436)
