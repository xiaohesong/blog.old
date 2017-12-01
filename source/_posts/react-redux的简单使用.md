---
title: react-redux的简单使用
date: 2017-12-01 15:31:17
tags:
  - redux
  - react
  - redux-thunk
---

正式接触react应该有两个月了,项目开始也有一个月了,
开始新项目的时候就没有打算使用redux,因为感觉学习这个的成本挺高.
这两天,项目第一版已经出来了,昨天下午有时间就抓紧时间看看redux相关的东西,大致有个了解,然后今天就照葫芦画瓢,把一个模块给整理了下,换成了redux.
在这里记录下使用,以备后需.

### react-redux
使用前得需要安装
```shell
npm i --save react-redux redux redux-thunk
```
1. 先创建actions
获取用户列表的相关操作.
```javascript
// src/actions/users/index.js
import * as usersTypes from '../../constants/users/ActionTypes';

export const queryUsersSuccess = users => ({ type: usersTypes.QUERY_USERS_SUCCESS, users })

export const queryUsers = () => {
  return dispatch => {
    MyFetch.get(`v1/users`).then(data => {
      dispatch(queryUsersSuccess(data.users))
    })
  }
}
```

```javascript
// MyFetch.js
import { message } from 'antd';
const API_URL = process.env.REACT_APP_DEV_API_URL
var Fetch = {
    get(path) {
        return new Promise((resolve, reject) => {
            fetch(`${API_URL}/${path}`, {
                headers: new Headers({
                    'token': localStorage.getItem("my-custom-token"),
                    'Accept': 'application/json',
                    'Content-Type': 'application/json',
                })
            }).then(res => {
                return handleStatus(res);
            })
                .then(json => {
                    resolve(json);
                })
                .catch(err => {
                    reject(err);
                });
        });
    },
    post(params) {},
    ...
}

function handleStatus(res) {
    let errors;
    switch (res.status) {
        case 200:
            return res.json();
        case 500:
            console.log("500错误");
            message.error('服务器内部错误', 5)
            errors = `${res.status}, ${res.statusText}`
            throw errors
        case 404:
            message.error("资源不存在", 5)
            errors = `${res.status}, ${res.statusText}`
            throw errors
        case 401:
            message.error("登录会话过期,请重新登录", 5)
            localStorage.removeItem("my-custom-token")
            window.location.href = '/login'
            break;
        case 403:
            message.error("无权限访问", 5)
            errors = `${res.status}, ${res.statusText}`
            throw errors
      default:
    }
}
```
2. 创建constants
```javascript
// src/constants/users/ActionTypes.js
export const QUERY_USERS = 'QUERY_USERS'
export const QUERY_USERS_SUCCESS = "QUERY_USERS_SUCCESS"
```
3. 创建 reducers相关
```javascript
// src/reducers/users.js
import {QUERY_USERS_SUCCESS} from "../constants/users/ActionTypes";

const initState = {
  users: []
}

export default function users(state = initState, action) {
    console.log("Welcome to reducer users");
    switch (action.type) {
        case QUERY_USERS_SUCCESS:
          return {
            ...state,
            users: action.users
          }
        default:
            return state
    }
}
```

```javascript
// src/reducers/index.js
import { combineReducers } from 'redux'

//BASE API
import users from './users'

const rootReducer = combineReducers({
  users,
})

export default rootReducer

```
这里创建了相关的reducers.

4. 在根组件store
```javascript
// src/index.js
import React from 'react';
import { Provider } from 'react-redux'
import configureStore from './configureStore';
import ReactDOM from 'react-dom';
import App from './App';
const store = configureStore()

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

```javascript
// src/configureStore.js
import {createStore, applyMiddleware} from 'redux';
import thunkMiddleware from 'redux-thunk';
import rootReducer from './reducers';

// let store = createStore(combineReducers);

export default function configureStore() {
    const store = createStore(
        rootReducer,
        applyMiddleware(thunkMiddleware)
    );

    return store;
}
```

5. connect
```javascript
// src/components/User.js
import * as UserActions from '../../actions/users';
...
class User extends Component{
  componentDidMount(){
       this.props.actions.queryUsers();
  }
}

const mapStateToProps = state => ({
  company: state.companies,
	users: state.users //这个是reducers定义的
})

const mapDispatchToProps = dispatch => ({
    actions: bindActionCreators(UserActions, dispatch)
})

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Company)
```

这样就可以使用啦.
#### [本文原文](https://github.com/xiaohesong/ums/wiki/Redux%E4%BD%BF%E7%94%A8%E6%80%BB%E7%BB%93#react-redux)

ps: 目前看来,这个方法很low. 在action写了请求接口的方法,觉得不是很规范,后需会改进.
