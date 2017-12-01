---
title: create-react-app按需加载以及部署
date: 2017-12-01 15:27:06
tags:
  - react
  - code splitting
---

闲来无事,就想着把前几天折腾的问题重新梳理一遍,加深印象.
[上面一篇文章](http://www.jianshu.com/p/7e4533ca5707)有提到升级`eject`的`cra`项目.为什么升级这个项目,就是想要折腾,就是想要使用到`webpack2`,还有一个好处是加深对`create-react-app`的理解.坏处不言而喻,折腾浪费生命浪费时间.
### 按需加载
- `AsyncComponent.js`

```es6
import React, { Component } from "react";

export default function asyncComponent(importComponent) {
    class AsyncComponent extends Component {
        constructor(props) {
            super(props);

            this.state = {
                component: null
            };
        }

        async componentDidMount() {
            const { default: component } = await importComponent();

            this.setState({
                component: component
            });
        }

        render() {
            const Component = this.state.component;

            return Component ? <Component {...this.props} /> : null;
        }
    }

    return AsyncComponent;
}
```
这个是异步加载组件的方法.到时在需要的组件上加入引用就好.例如在路由里.
- `Menu.js`

```es6
const AsyncUser = asyncComponent(() => import("./User"));
...
<Route path="/users" component={AsyncUser}/>
```

### 部署(nginx以及npm2)
- nginx
    - 准备工作
    ```shell
        npm run build
    ```
    - `sudo vim /etc/nginx/sites-enabled/react-app.conf`
   ```nginx
      server {
          listen 7878;
          server_name 127.0.0.1;
          root /home/deploy/workspace/project/build;
          location / {
              try_files $uri /index.html;
              add_header   Cache-Control no-cache;
              #expires      1d;
          }
      }
   ```
    - 启用nginx配置
    ```shell
        sudo service nginx reload
        sudo service nginx restart
    ```
  - 注意
  如果是阿里云服务器,可能访问失败,阿里云的安全策略默认是只开启了`80`端口,首先去实例里检测其他的端口是否开放.

- pm2
  - `installl and start`
      ```shell
          sudo npm install pm2 -g
          pm2 start npm -- start
      ```

  - `nginx`
      ```shell
        # sudo vim /etc/nginx/sites-available/react-app.conf
        server{
              listen 80 default_server;
              server_name YOUR-SERVER-NAME;
              location / {
                proxy_pass http://localhost:3000; #or any port number here
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
              }
        }
      ```

   - `restart`
     ```shell
      sudo service nginx reload
      sudo service nginx restart
    ```

### 参考
  - [Code Splitting 参考](https://serverless-stack.com/chapters/code-splitting-in-create-react-app.html)
