---
title: React和Rails
date: 2017-03-05 15:37
tags:
  - react
  - rails
categories:
  - 迁移
---

对于rails中集成react组件有两个比较好的gem可供选择。

### 1. [react_rails](https://github.com/reactjs/react-rails)
    这个gem相对简单一点,有一个很好的例子可供参考。
   [reactjs-a-guide-for-rails-developers](https://www.airpair.com/reactjs/posts/reactjs-a-guide-for-rails-developers)

### 2. [react_on_rails](https://github.com/shakacode/react_on_rails)

这个gem复杂些，对环境要求比较高。

[环境要求](https://github.com/shakacode/react_on_rails/blob/master/docs/tutorial.md)

1. [nodejs](https://github.com/creationix/nvm#install-script)
2.	[yarn](https://yarnpkg.com/lang/en/docs/install/#linux-tab)
3.	foreman  -- gem install foreman

[recet_on_rails example](https://github.com/shakacode/react-webpack-rails-tutorial)

本人使用react_rails和react_on_rails两个gem实现了同一个例子。
react_rails是使用coffee编写。
react_on_rails是使用原生的jsx编写。
[源码](https://github.com/xiaohesong/bill)

react_rails主文件在	` app/assets/javascripts/components` 下

react_on_rails主文件夹在
`client/app/bundles/HelloWorld/components/records`下。


该如何选择:
简单的使用react_rails.
复杂的使用react_on_rails

### 3. 纯前端的React Deomo
- 主要是实现了一个简单的TODO功能。
- UI是从bootstrsap切换成了Ant-Design.
- [react源码](https://github.com/xiaohesong/react_blog)，wiki中有介绍如何接入ant-design.其实官方文档里已经说明了如何使用。
- [Ant-Design官网](https://ant.design/docs/react/introduce)

## [react jsx语法规范](https://github.com/airbnb/javascript/tree/master/react)

## Note

1.	每次在组件中调用方法，在执行该方法时会报错找不到this(null),这个时候需要去手动绑定，在构造函数的函数里。

```js
	constructor(props) {
    super(props);
    this.state = {
	     name: 'xiaozhu'
    }
    this.example = this.example.bind(this);
  }

　　example() {
		// do somthing
	}
```
这样很麻烦吧，可以换成es6的写法：

```js
constructor(props) {
    super(props);
    this.state = {
	     name: 'xiaozhu'
    }
    //this.example = this.example.bind(this);
  }

　　example = (param) => {
		// do somthing
	}
```

这样写，就不需要自己去手动绑定了。
[参考](https://facebook.github.io/react/docs/handling-events.html)
具体的不是es6的效果请[查阅](https://facebook.github.io/react/docs/react-without-es6.html)
２．　组件之间进行通信，需要在对应的组件上添加属性。

	// OneComponent
	render(){
		return(
			<ExampleComponent handleOperation={this.doSomthing}>
		)
	}

	doSomthing = (param) => {
		// do somthing
	}


	// ExampleComponent
	render(){
		return(
			<h1>Hi,Xiaozhu</h1>
			<a className='btn btn-default' onClick={this.handleToUpdate}>
            Update
          </a>
		)
	}

	handleToUpdate = （e） => {
		var _this = this
		e.preventDefault();
		$.ajax({
			...
			success: function(data){
				_this.props.handleOperation (data)
			}
		})
	}
