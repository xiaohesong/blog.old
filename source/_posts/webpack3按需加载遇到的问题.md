---
title: webpack3按需加载遇到的问题
date: 2017-12-01 15:36:34
tags:
  - react
  - webpack
---

这节主要说的是自定义`webpack`之关于配置按需加载遇到的问题
- `AsyncComponent.js`
```javascript
import React, {Component} from "react";

export default function asyncComponent(importComponent) {
  class AsyncComponent extends Component {
    constructor(props) {
      super(props);

      this.state = {
        component: null
      };
    }

    async componentDidMount() {
      const {default: component} = await importComponent();

      this.setState({component: component});
    }

    render() {
      const Component = this.state.component;

      return Component
        ? <Component {...this.props}/>
        : null;
    }
  }

  return AsyncComponent;
}
```

2. `Home.js`

```javascript
import React from 'react'
const Home = () => {
  return (<h1>It is My Home</h1>)
}

export default Home
```

3. `Menu.js`
```javascript
import React from 'react'
import {Menu, Icon} from 'antd';
import {Link} from 'react-router-dom'
import asyncComponent from './AsyncComponent';

const AsyncHome = asyncComponent(() => import("./Home"))
const SubMenu = Menu.SubMenu;

class MyMenu extends React.Component {
  render() {
    return (
      <Router>
        <Route path="/" component={AsyncHome} />
        <Menu theme="dark" defaultSelectedKeys={['1']} mode="inline">
          <Menu.Item key="2">
            <Icon type="desktop"/>
            <Link to="/">
              <span>Option 2</span>
            </Link>
         </Menu.Item>
        </Menu>
      </Router>
    )
  }
}

export default MyMenu
```

预期的结果,这样应该是可以的.可是对于一些`es6`语法,`webpack`还是需要配置一番.所以碰到了问题.

### 发现的问题
1. `Uncaught ReferenceError: regeneratorRuntime is not defined`

解决方法: 引入`transform-runtime`.

[包的地址](https://www.npmjs.com/package/babel-plugin-transform-runtime)

[transform-runtime与babel-polyfill的区别](https://segmentfault.com/q/1010000005596587?from=singlemessage&isappinstalled=1)

2. `Uncaught ReferenceError: webpackJsonp is not defined`

解决方法: 引入`CommonsChunkPlugin`

[custom-react源码](https://github.com/xiaohesong/custom_react)

# [原文地址](https://github.com/xiaohesong/ums/wiki/webpack%E9%81%87%E5%88%B0%E7%9A%84%E4%B8%80%E4%BA%9B%E9%97%AE%E9%A2%98(2))
