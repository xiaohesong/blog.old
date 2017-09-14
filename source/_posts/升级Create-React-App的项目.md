---
title: 升级Create React App的项目
date: 2017-09-14 13:38:52
tags:
  - react
  - create-react-app
categories:
  - react
---

为了实现路由按需加载,因为触及到版本的相关问题([版本详情](https://github.com/xiaohesong/react_blog/wiki/code-splitting)),所以本着折腾的原则,想去尝试升级一下,也是为了以后实践项目多知道几个坑吧.

情况是这的,之前也不知道版本信息的要求,直接在电脑上开搞了.用的官方的`create-react-app`.
- 升级前的相关环境
  1. `node -v` => 4.几的忘记了
  2. `npm -v` => 3.几的忘记了.
  3. `webpack version` => 1.14.0
  4. `react-router-dom` => 4.2.2
  5. `react-scripts` => 0.9.x

- 升级后的环境.
  1. `node -v` => 6.1.0
  2. `npm -v` => 4.6.1
  3. `webpack version` => 3.5.6
  4. `react-router-dom` => 4.2.2
  5. `react-scripts` => 1.0.x(3)

咋说呢,其实升级也还好,[按照官方的来就好了](https://github.com/facebookincubator/create-react-app/releases).

但是,因为项目里使用了antd.三月份的时候,想接触学习`react`,就写了一个`Todo`并引入了`ant-design`.搁浅了几个月,这几天要做一个权限管理的`demo`. 所以现在的这个`demo`就按照之前的那个配置来了.那个时候是需要暴露出配置去修改.[现在官方文档已经修改了](https://ant.design/docs/react/use-with-create-react-app-cn).
因为`eject`操作是不可逆的,而且作为一个新手,也是尽量避免暴露配置文件的.

可以发现`create-react-app`的升级文档也有强调,未`eject`的情况下.并且也在`issue`里说明了,不会提供任何帮助如果你暴露出来了配置.

> Note that we don't provide help for webpack questions after ejecting.
    You can find webpack docs at https://webpack.js.org/.

#### 升级操作.
- 备忘
- 切换分支

  切换到一个新的分支去处理升级情况.

- 拷贝现有配置,以防万一.
  ```shell
    cp package.json backup-package.json
    cp yarn.lock backup-yarn.lock
  ```

- 替换需要的版本

  找到你需要的版本的升级说明,去初始化一个新的项目(example).[我需要的版本是1.0.0](https://github.com/facebookincubator/create-react-app/releases/tag/v1.0.0)
  ```bash
    npm install --save-dev --save-exact react-scripts@1.0.0
    npm install -g create-react-app
    create-react-app example
  ```
  初始化项目之后,就有一个现成的你需要的版本了.
  我们来替换一些相关的配置信息.
  - `config`文件夹
  由于我的暴露出了配置信息,所以我需要把`~/my_project/config`给替换成新的版本配置.
  ```bash
    rm -rf ~/my_project/config
    cp -r ~/example/config/ ~/my_project/
  ```

  - `scripts`文件夹
  ```bash
    rm -rf ~/my_project/scripts
    cp -r ~/example/scripts/ ~/my_project/
  ```

  - `package.json`
  这个是`npm init`的产物.
  ```bash
    cp ~/example/package.json ~/my_project/
  ```
  注意,`package.json`里有一个地方需要改动,就是`name`,换成你的项目的名字.比如 `example => my_project`.

- 重新安装.
这里就基本把项目配置换成了新的,那么就需要重新安装相关的插件.
```bash
cd ~/my_project
rm -rf node_modules
npm cache clean
npm install
```
这里就把相关的重新安装了,那么我们也需要把项目之前的第三方库安装上去.
我的比较简单,只有`react-router-dom`,`antd`,`webpack`.

- 番外
  - 遇到的问题
    - `Import in body of module; reorder to top import/first`
       我的这个错误主要是`css`文件的问题.
       ```es6
         import A from '../A.css'
         import A from '../A' // A.js
         //改成下面这样
         import A from '../A' // A.js
         import A from '../A.css'
       ```
       `ESLint`的原因.

    - 法子引入`antd`
      现在的官网的方式没法子引入`antd`,或许是因为暴露出了配置的原因吧.
      使用的依然是`eject`之后的法子.[稍有变动](https://github.com/xiaohesong/react_blog/wiki/How-To-Use-Ant-Design)



#### [原文链接](https://github.com/xiaohesong/react_blog/wiki/Upgrade-Create-react-app)
