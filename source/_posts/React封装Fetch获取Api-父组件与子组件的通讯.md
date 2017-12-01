---
title: 'React封装Fetch获取Api, 父组件与子组件的通讯'
date: 2017-12-01 15:28:12
tags:
  - fetch
  - react
---

一个项目下来,和后台数据的交互实在是太多了,如果不封装一下就感觉怪怪的,而且也很不DRY.封装起来之后,会省事很多很多.

- [封装fetch](#封装fetch)
- [父子组件的通讯](#父子组件的通讯)

### 封装Fetch
```es6
// MyFetch.js
const API_URL = process.env.REACT_APP_DEV_API_URL
var methods = {
	get(path) {
      return new Promise((resolve, reject) => {
          fetch(`${API_URL}/${path}`,{
              headers: new Headers({
                  'my-token': "xxxxx...",
                  'Accept': 'application/json',
                  'Content-Type': 'application/json',
              })
          }).then(res => {
            if (res.ok) {
                return res.json();
            } else if(res.status === 500) {
              let errors = `${res.status}, ${res.statusText}`
              throw errors
            } else if (res.status === 404) {
              let errors = `${res.status}, ${res.statusText}`
              throw errors
            }else if (res.status === xxx) {

            }
          })
          .then(json => {
              resolve(json);
          })
          .catch(err => {
              reject(err);
          });
      });
    },
}
export default methods
```

调用的时候也很简单.
```es6
import MyFetch from './MyFetch';
MyFetch.get(users).then(data => {
  console.log("respond data is", data)
})
```
这样就可以在Get请求的时候统一调用该接口,进行了一次封装.


### 父子组件的通讯
- 父组件传递数据到子组件.
```es6
import React, {Component} from 'react';

export default class Parent extends Component{
	render() {
		return(
			<Child name={'issue'} age='1'/>
		)
	}
}

export default class Child extends Component {
	constructor(props){
	    super(props);
	    console.log(this.props.name, this.props.age)
	    //会获得对应的数据.('issue', '1')
	}
}
```

- 子组件传递数据到父组件

```es6
import React, {Component} from 'react';

export default class Parent extends Component{
	render() {
		return(
			<Child name={this.name} />
		)
	}

	name = (value) => {
		console.log("receive name value is", value)
	}
}

export default class Child extends Component {
	render(){
		return(
			<input onChange={this.handleChange} />
		)
	}

	handleChange = (e) => {
		let value = e.target.value
		this.props.name(value)
	}
}
```
这样就可以把获取到的用户输入的内容传递给父组件了.

#### 需要注意: 父组件加载子组件,子组件的构造函数只会执行一次.
如果父组件(容器组件)获取远程数据,传递到子组件,不要在子组件的状态里写入数据.直接使用props.data去获取.
举例如下:
```es6
import React, {Component} from 'react';

export default class Parent extends Component{
	state = {
		users: []
	}

	componentDidMount() {
		MyFetch.get('users').then(data => {
			this.setState({
				users: data.users
			})
		})
	}

	render() {
		return(
			<Child dataSource={this.state.users} />
		)
	}
}

export default class Child extends Component {
	constructor(props){
		super(props);
		this.state = {
			users: this.props.users
		}
	}
	render(){
		console.log(this.state.users) #[]
		console.log(this.props.users) #[{user1},{user2},...]
		return(
			<table>
				xxx
			</table>
		)
	}
}
```
出现上面这种情况是因为父组件在获取到数据之前就已经加载了子组件.但是子组件在一个生命周期,构造函数只会执行一次.所以父组件改变状态之后,是无法再次执行子组件构造函数的内容.

#### [Github原文](https://github.com/xiaohesong/ums/wiki/React%E5%B0%81%E8%A3%85Fetch%E8%8E%B7%E5%8F%96Api,-%E7%88%B6%E7%BB%84%E4%BB%B6%E4%B8%8E%E5%AD%90%E7%BB%84%E4%BB%B6%E7%9A%84%E9%80%9A%E8%AE%AF#%E5%B0%81%E8%A3%85fetch)
