---
title: 'react-thunk迁到redux-saga,他们的对比'
date: 2017-12-01 15:33:02
tags:
  - redux
  - react
  - redux-saga
---


### thunk-saga
背景: 刚开始学习前端以及react.之前粗略的对比了下`thunk`以及`saga`.发现`thunk`与`saga`总体差不多,对我来说都够用,再考虑到学习成本,我还是选择使用了thunk. 但是使用thunk重构几个模块之后发现登录流程很麻烦,需要`promise`或者`async/wait`的支持才可以很好的完成登录流程,我的解决办法是在回调里调用(尝试过`async/promis`不可以,里面的步骤比较繁琐),这个很low逼,我也很不喜欢,以后维护起来会很吃力,所以决定切换到saga.

两者的对比,先从简单的获取数据说起(获取用户列表)
一: 大体相同的部分
```es6
...
import * as actions from '../actions/users
import {connect} from 'react-redux';
import { bindActionCreators } from 'redux'
class User extends React.Component {
  ...

}
const mapStateToProps = (state, ownProps) => ({
  users: state.users,
})

const mapDispatchToProps = dispatch => ({
    actions: bindActionCreators(actions, dispatch)
})

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(User)
```
上面这部分就是大致相同的.

二: 不同点
- 配置不同
 saga的store配置如下

```javascript
// ./configureStore.js
import {createStore, applyMiddleware} from 'redux';
import thunkMiddleware from 'redux-thunk';
import rootReducer from './reducers';

import createSagaMiddleware from 'redux-saga';
import rootSaga from './sagas';
const sagaMiddleware = createSagaMiddleware()
// let store = createStore(combineReducers);

export default function configureStore() {
    const store = createStore(
        rootReducer,
        applyMiddleware(sagaMiddleware)
    );

    sagaMiddleware.run(rootSaga)

    return store;
}
```
可以发现初始化的时候就去运行 saga.

我们来看看saga:

```javascript
// ./sagas/index.js
import {watchFetchUsers} from './users';

function* rootSaga() {
  /*The saga is waiting for a action called LOAD_DASHBOARD to be activated */
  yield all(
    [
      fork(watchFetchUsers),
    ]
  )
}

export default rootSaga
```
我们来看看watchFetchUsers

```javascript
import {put, takeEvery, call} from 'redux-saga/effects';
import {queryUsers} from './Api';
import * as actions from '../../actions/users';
import {GET_USER_REQUEST} from '../../constants/users/ActionTypes'

export function * watchFetchUsers() {
  yield takeEvery(GET_USER_REQUEST, getUsers)
}

export function* getUsers() {
  const data = yield call(queryUsers);
  yield put(actions.queryUsersSuccess(data.users));
}
```

```javascript
// ./Api
import MyFetch from '../../tools/MyFetch'

export const queryUsers = () => {
  return MyFetch.get(`v1/users`);
};
```
可以发现,saga里有 `watchFetchUsers`和`getUsers`.我们在`rootSagas`里是有`fork`这个`watchFetchUsers`的.然后通过`watchFetchUsers`去触发`getUsers`. 那么如何触发`watchFetchUsers`呢?我们需要改变下用户的`actions`.

```javascript
// actions/users/index.js
import * as types from constants/users/ActionTypes
export const getUserRequest = () => ({type: types.GET_USER_REQUEST})
```
现在我们有了这个`action`, 那么我们就可以去使用他发起一个请求.

```javascript
// components/User.js

class User extends React.Component {
  ...
  componentDidMount() {
    this.props.actions.getUserRequest()
  }
}
```

这样子他就会去执行`getUserRequest`方法,这样就会被`watchFetchUsers`给监听到,再去通过`type`(GET_USER_REQUEST)去匹配`getUsers`方法.
再`getUsers`方法最后有 `yield put(actions.queryUsersSuccess(data.users));`这个`put`就是相当于thunk的`dispatch`.

写了一天之后给我的感觉就是: thunk需要你自己去匹配需要的动作,saga是写一个监听方法,他自己去分发对应的action.
或许我这样的写法是不规范的,但是我还是决定先切换到`saga`

#### 下面是一个关于登录发送短信倒计时的对比.

1. thunk
```javascript
export const sendAuthCodeToPhone = (self, phone) => {
    return dispatch => {
      dispatch(sendCodeStart())
      MyFetch.post(`v1/verification_code`, {phone: phone}).then(data => {
        if(data.status === 200){
          dispatch(snedCodeSuccess())
          self.timer = setInterval(() => {
            dispatch(tick(self))
          }, 1000);
        }else {
          dispatch(snedCodeFail())
          message.error(data.message)
        }
      })
    }
}

const tick = (self) => {
  return dispatch => {
    let counter = self.props.login.count
    if (counter < 1) {
      clearInterval(self.timer)
      dispatch(timerStart())
    } else {
      dispatch(timerStop(counter))
    }
  }
}
```

2. saga

```javascript
import { eventChannel, END } from 'redux-saga'
import {put, takeEvery, call, take, fork, takeLatest} from 'redux-saga/effects';

export function* sendCode(action) {
  let {self} = action
  const result = yield call(sendCodeToPhone, action)
  if(result.status === 200){
    yield put(actions.snedCodeSuccess())
    const timeChannel = yield call(tick, self)
    try {
      while (true) {
        // take(END) will cause the saga to terminate by jumping to the finally block
        let seconds = yield take(timeChannel)
        yield put(self.props.actions.timerStop(seconds))
      }
    } finally {
      yield put(self.props.actions.timerStart())
    }
  }else {
    yield put(actions.snedCodeFail())
    message.error(result.message)
  }
}

function tick(self) {
  return eventChannel(emitter => {
      const timer = setInterval( function() {
        let counter = self.props.login.count
        if (counter < 1) {
          emitter(END)
          clearInterval(timer)
        } else {
          emitter(counter)
        }
      }, 1000);

      return () => {
        clearInterval(timer)
      }
    }
  )
}
```
