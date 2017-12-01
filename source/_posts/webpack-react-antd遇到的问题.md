---
title: webpack react antd遇到的问题
date: 2017-12-01 15:35:17
tags:
  - webpack
  - react
---

1. [初始化项目文件](#初始化项目文件)
2. [加入antd](#加入antd)
3. [问题](#问题)
4. [总结](#总结)
5. [相关链接](#相关链接)

最近这两天把`redux`切换到了`redux-saga`之后,就想学习学习`webpack`. 咋说呢,这个东西被大家说的神乎其神,所以在我的认知里还是蛮神秘的(新手的感觉而已,不喜勿喷).
今天上午把webpack看了下,有个大致的方向,就想配一个简单的,可以本地开发运行的`react`([github custom_react](https://github.com/xiaohesong/custom_react))应用.

### 初始化项目文件
- `package.json`
这个很简单,直接`npm init`就好.  [npm?](https://github.com/xiaohesong/ums/wiki/environment)

- `index.html`
我们新建一个`index.html`.
```shell
mkdir -p public
cd public
touch index.html
```
`index.html`内容如下:
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="theme-color" content="#000000">
    <title>custom</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

- `index.js`
开始这个js文件之前,我们需要使用相关的包
```shell
npm i --save react react-dom
```
然后我们的`index.js`是这样的.
```javascript
import ReactDOM from 'react-dom';
import App from './App';
import React from 'react'

ReactDOM.render(
    <App/>,
    document.getElementById('root')
);
```

- `App.js`
然后我们的`App.js`是这样的.
```javascript
import React, { Component } from 'react';

class App extends Component {
    render() {
        return (
            <h1>Hello Word</h1>
        )
    }

}

export default App
```

- `webpack.config.js`
同样,在开始新建文件之前, 先安装相关的包.
```shell
npm i --save webpack 或者 npm i webpack -g
npm install --save-dev webpack-dev-server // webpack的服务器
npm install --save-dev babel-core babel-loader  babel-preset-es2015 babel-preset-react babel-preset-stage-0
npm install --save-dev css-loader postcss-loader style-loader
npm install --save-dev autoprefixer
```
> `-g`是全局安装,执行打包命令的时候直接`webpack`就好了.如果是`--save`安装,就需要打包的时候使用 `node_modules/.bin/webpack`.在这里,我们安装了`webpack`以及`bable`家族.(利用bable去转换es6,去转换jsx, ...).还有`css`相关的处理.

```javascript
// webpack.config.js
var webpack = require('webpack');
var path = require('path')
module.exports = {
    entry: ['webpack/hot/dev-server', __dirname + "/index.js"],
    output: {
        path: __dirname + "/build",
        filename: "bundle.js",
        publicPath: "/assets/"
    },
    module: {
        loaders: [
            {
                test: /\.(js|jsx)$/, // test 匹配js和jsx）
                exclude: /node_modules/, //屏蔽不需要处理的文件
                loader: 'babel-loader',
                query: {
                    "presets": [
                        "react",
                        "es2015",
                        "stage-0"
                    ],
                }
            },
            {
                test: /\.css$/,
                loader: 'style-loader!css-loader?importLoaders=1',
            }
        ]
    },
    plugins: [
        new webpack.HotModuleReplacementPlugin(), //模块的热替换插件
    ],
    devServer: {
        contentBase: path.join(__dirname, "/public"), // index.html的位置
        historyApiFallback: true,
        inline: true,
        port: 3008, //这里写你自己想要的启动端口
        compress: true,
        progress: true,
    }
}
```
好的,目前我们的`webpack.config.js`大致就是这样,现在我们还需要修改`package.json`去运行他.
```javascript
// package.json
. . .
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "start": "webpack-dev-server"
  },
. . .
```
现在我们可以在终端输入命令尝试.
```shell
npm run build //会去构建配置,会有文件output.
npm start // 启动项目.
```
请确定我们的`index.html`引入了文件.
```html
. . .
<body>
    <div id="root"></div>
    <script src="/assets/bundle.js"></script>
 </body>
. . .
```
不出意外,正常启动,输入`localhost:3008`之后,就会出现`Hello Word!`

### 加入`antd`
首先安装相关的包
```shell
npm install antd --save
npm install babel-plugin-import --save-dev
```
然后在我们的`webpack.config.js`配置使用.
```javascript
// webpack.config.js
. . .
query: {
                    "presets": [
                        "react",
                        "es2015",
                        "stage-0"
                    ],
                    plugins: [
                        ["import", {
                            libraryName: "antd",
                            style: "css"
                        }],
                    ]
                }
. . .
```
然后我们尝试是否引入了`antd`.
- 修改`App.js`
```javascript
import React, { Component } from 'react';
import MainLayout from './src/MainLayout'

class App extends Component {
    render() {
        return (
            <MainLayout />
        )
    }

}

export default App
```

- src/MainLayout.js
```javascript
import React from 'react'
import { Layout, Menu, Breadcrumb, Icon } from 'antd';
const {Header, Content, Footer, Sider} = Layout;
const SubMenu = Menu.SubMenu;


class SiderDemo extends React.Component {
    state = {
        collapsed: false,
    }


    onCollapse = (collapsed) => {
        console.log(collapsed);
        this.setState({
            collapsed
        });
    }

    render() {
        return (
            <Layout style={{
                minHeight: '100vh'
            }}>
        <Sider
            collapsible
            collapsed={this.state.collapsed}
            onCollapse={this.onCollapse}
            >
          <div className="logo" />
          <Menu theme="dark" defaultSelectedKeys={['1']} mode="inline">
            <Menu.Item key="1">
              <Icon type="pie-chart" />
              <span>Option 1</span>
            </Menu.Item>
            <Menu.Item key="2">
              <Icon type="desktop" />
              <span>Option 2</span>
            </Menu.Item>
            <SubMenu
            key="sub1"
            title={<span><Icon type="user" /><span>User</span></span>}
            >
              <Menu.Item key="3">Tom</Menu.Item>
              <Menu.Item key="4">Bill</Menu.Item>
              <Menu.Item key="5">Alex</Menu.Item>
            </SubMenu>
            <SubMenu
            key="sub2"
            title={<span><Icon type="team" /><span>Team</span></span>}
            >
              <Menu.Item key="6">Team 1</Menu.Item>
              <Menu.Item key="8">Team 2</Menu.Item>
            </SubMenu>
            <Menu.Item key="9">
              <Icon type="file" />
              <span>File</span>
            </Menu.Item>
          </Menu>
        </Sider>
        <Layout>
          <Header style={{
                background: '#fff',
                padding: 0
            }} />
          <Content style={{
                margin: '0 16px'
            }}>
            <Breadcrumb style={{
                margin: '16px 0'
            }}>
              <Breadcrumb.Item>User</Breadcrumb.Item>
              <Breadcrumb.Item>Bill</Breadcrumb.Item>
            </Breadcrumb>
            <div style={{
                padding: 24,
                background: '#fff',
                minHeight: 360
            }}>
              Bill is a cat.
            </div>
          </Content>
          <Footer style={{
                textAlign: 'center'
            }}>
            Ant Design ©2016 Created by Ant UED
          </Footer>
        </Layout>
      </Layout>
        );
    }
}

export default SiderDemo
```
然后我们`npm start`一下,应该是可以的了.

### 问题
怎么说呢,其实这些东西都是可以找到的,但是因为版本等原因,难免会碰到一些小问题,那么我今天就凯说这半天我碰到的两个问题.
- `Cannot GET /`

这个问题倒是还好,被我猜测到了,是没有运行起来,应该是说没有找到`index.html`文件,所以把服务器当前工作地址指向`index.html`坐在的文件夹下. 我们这里是在`public`文件夹下.

- 页面空白

页面空白,不显示`Hello Word`. 打开浏览器`console`发现是没有找到对应的`bundle.js`. 去`sources`里也没有对应的`js`文件.这个也有人碰到过,[看这个issue](https://github.com/webpack/webpack-dev-server/issues/645)
所以,我们需要指定一下`publicPath`, 否则页面是找不到的.

- `es6`语法报错

出现`Module Build Faild`相关的错误.就是`es6`的语法错误.找了很久都不行,最后在一篇外文里看到了解决办法(最后会放上链接).加上`stage-0`的支持就好了.

### 总结
目前碰到的问题就是那么多,而且这个文章只适合一点点也不懂的朋友,还有很多东西需要去处理,比如生产模式,性能优化..,所以,还得一起继续学习.

### 相关链接
- [github 源码](https://github.com/xiaohesong/custom_react)
- [github 原文](https://github.com/xiaohesong/ums/wiki/webpack%E9%85%8D%E7%BD%AE%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98)
- [React and ES6 - Part 6, React and ES6 Workflow with Webpack](http://egorsmirnov.me/2016/04/11/react-and-es6-part6.html)
