---
layout: post
title:  "react flux浅析"
date:   2015-09-26 12:15:09
categories: tech
---


上周参加velocity大会,fb的前端负责人分享了`FLUX`这种新的架构模式,回来仔细研究了下Flux.

## flux是什么?
首先看下flux的基本模型,
![img](http://gtms03.alicdn.com/tps/i3/TB1QumCJXXXXXacXVXX9RKLWVXX-1280-284.jpg)
Action就是用户与view层之间的交互行为,Action会触发Dispatcher,Dispatcher通过注册的回调函数把这个动作分发给对应的store.store在处理完数据以后会触广播一个"change"事件通知react组件，react组件会监听这些"change"事件并且从store暴露的函数中获取数据，以todolist为例描述上述过程:

- 1.Header组件中触发dispatcher

```
//react componets
var Header = React.creatClass({
	render:function(){
		return (
			<input onSave={this._onSave}>
		)
	},
	_onSave:function(text){
		//触发dispatcher
		TodoActions.create(text)
	}
})
```
- 2.dispatch分发给注册的回调传递动作,这里的`动作`实际就是一个对象，它包含数据(payload)与类型(actionType)两部分

```
//TodoActions
var TodoActions = {
  create: function(text) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_CREATE,
      text: text
    });
  }
  }
```

- 3.在回调中处理数据,并广播消息,这是使用Nodejs自带的Event模块

```
var EventEmitter = require('events').EventEmitter;
var emitter = new EventEmitter();

var creat = function(){
	....
}
//TodoStore
var TodoStore = {
	getAll:function(){
		...
	},
	emitChange: function() {
    	emitter.emit(CHANGE_EVENT);
  	},
  	addChangeListener: function(callback) {
    	emitter.on(CHANGE_EVENT, callback);
  }
}
AppDispatcher.register(function(action){
	switch(action.actionType){
	 case TodoConstants.TODO_CREATE:
      text = action.text.trim();
      if (text !== '') {
        create(text);
        //广播
        TodoStore.emitChange();
      }
      break;
	}
})
```
- 4.在Todo组件中监听TosoStore的change事件，并从store暴露的方法获取新数据,然后更新view

```
var TodoApp = React.creatClass({
	getInitialState:function(){
		return{
		 allTodos:TodoStore.getAll()
		}
	},
	componentDidMount: function() {
		//监听
    	TodoStore.addChangeListener(this._onChange);
  	},
  	render:function(){
  	 ...
  	}
})

```

通过上面代码可以看到,FLUX它是一种控制数据单向流动的架构模式,实现 这种思想有两个至关重要的东西，一是`Dispatch`，另一个是上面第三步用到node事件模块(events.EventEmitter),如果要是熟悉`发布订阅`设计模式的话其实这两者都很好理解.

## Dispatch

首先我们看下dispatch的[源码](https://github.com/facebook/flux/blob/master/dist/Flux.js).dispatch提供了5个方法:`register`,`unregister`,`waitFor`,`dispatch`,`isDispatching`,这里register与dispatch就是类似我们常见的订阅与发布,不同的是dispatch执行的时候会将参数分发给所有注册的回调函数，而传统的pub/sub会指定特定的订阅类型.

```
Dispatcher.prototype.dispatch = function dispatch(payload) {
!!this._isDispatching ? true ? invariant(false, 'Dispatch.dispatch(...): Cannot dispatch in the middle of a dispatch.') : invariant(false) : undefined;
	this._startDispatching(payload);
	try {
	//循环所有的回调
		for (var id in this._callbacks) {
			if (this._isPending[id]) {
			continue;
		}
		this._invokeCallback(id);
	}
	} finally {
		this._stopDispatching();
	}
}
```
register只是一个简单的订阅,它会返回一个id,

```
Dispatcher.prototype.register = function register(callback) {
var id = _prefix + this._lastID++;
this._callbacks[id] = callback;
return id;
}
```
register返回的id一般只在waitFor中使用,waitFor可以指定回调的执行顺序,`在指定的回调函数执行之后才执行当前回调`.

## Store
store的作用与传统的model有点类似,主要用作数据处理,

```
var EventEmitter = require('events').EventEmitter;
var emitter = new EventEmitter();

//TodoStore
var TodoStore = {
	getAll:function(){
		...
	},
	emitChange: function() {
    	emitter.emit(CHANGE_EVENT);
  	},
  	addChangeListener: function(callback) {
    	emitter.on(CHANGE_EVENT, callback);
  }
}
```
上面代码我们用EventEmitter来监听/广播事件,view组件基于这些事件来更新.

## controller-views
controler-view其实就是react组件,它监听着stores广播的事件,然后从stores中获取数据并且传递这些数据的到它的子组件中,例如

```
var TodoApp = React.createClass({

  getInitialState: function() {
    return getTodoState();
  },

  componentDidMount: function() {
    TodoStore.addChangeListener(this._onChange);
  }
  
  })
```

## 小结
目前我还没有在业务中应用过flux这种模式,对于有些细节还不是很清楚,比如它与MVC具体的差异在哪里?waitFor如果是异步队列应该怎么办?在项目具体实践之后才会更加深入了解flux.
