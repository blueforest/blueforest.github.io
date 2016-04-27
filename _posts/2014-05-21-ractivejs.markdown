---
layout: post
title:  "ractivejs简介"
date:   2015-05-21 12:15:09
categories: tech
---

![img](/assets/images/ractive.png)

### 为什么要用Ractive.js?
日常项目中工具类产品页面交互复杂,仅仅基于jQuery的开发方式已经不能满足界面复杂交互的要求.在新项目的前端开发中采用了[Ractive.js](http://www.ractivejs.org/)这个简单却功能强大的JS库,它实现了数据绑定、DOM实时更新、模板、事件处理等多个有用的功能,它专注于UI层,学习成本低.现有关于前端MVC方面的框架很多,例如angularjs、backbone等,他们提供了一整套的解决方案,比如路由、模板、数据双向绑定的,但是这些框架对于后端同学而言学习成本太高(比如说要用angular的话你就要明白什么是依赖注入、scope、directive这些概念),不符合现在与后端开发共建产品的实际情况.下面简单的介绍ractive的几个非常重要的特性,希望能帮大家快速入门.

### 一.hello world
- 首先创建一个```index.html```,引入```ractive.js```,并添加一个容器元素来渲染模板:

```html
<body>
    <div id='container'></div>
    <script src='Ractive.js'></script>
</body> 
```

- 编写模板并实例化一个Ractive对象

```html
    <div id='container'></div>
    
    <script id='template' type='text/ractive'>
    	<h1>{{name}}</h1>
    </script>
    <script src='ractive.js'></script>
    <script>
    var ractive = new Ractive({
     	el: 'container',
    	template: '#template',
    	data: { name:"Hello world"}
	});
    </script>

```

具体效果见[demo](http://jsfiddle.net/84xp4jan/14/),Ractive.js使用的模板遵循[mustache](https://mustache.github.io/)语法.

### 二.数据的更新
ractivejs数据的更新类似set、get存取器,通过set来更新数据，通过get来获取数据,例如上面Helloworld的例子中,我们打开控制台输入``` ractive.set('name',"alibaba")```,你会看到页面上会立即更新,同时我们通过```ractive.get('name')```可以获取当前```name```的值.看一个简单的例子([demo](http://jsfiddle.net/84xp4jan/15/)):

```javascript
var list = [{name:"Jim"},{name:"LUCY"},{name:"LILY"}]
var listView = new Ractive({
    el: 'container',
    template: '#template',
    data: { list:list}
});
    //修改list第一个值为sone
  listView.set('list.0.name','sone');
  //获取修改后的值
  console.log( listView.get('list') );
  
```

这里通过```set('list.0.name')```来动态修改数组第一个元素的name值.```list.0```表示数组的第一个元素.

### 三.数据的双向绑定
理解并掌握数据的双向绑定可以很大程度上提高我们的开发效率,不用再写那些繁琐的DOM操作代码.[demo](http://jsfiddle.net/84xp4jan/17/):

```html
  <div><h2>订阅报警状态:</h2>
  <label><input type='radio' name='{{status}}' value='Error' checked> red</label>
  <label><input type='radio' name='{{status}}' value='Warning'> green</label>
  <label><input type='radio' name='{{status}}' value='Critical'> blue</label>
  <p>当前报警状态: {{status}}</p>
</div>
```

上面代码为模板代码,通过简单模板我们就可以实现数据的双向绑定,将数据(javascript对象)映射到view层,界面的操作都会同步更新数据.这里我们可以通过```listView.get('status')```获取当前选中状态的值,同样可以通过set方法设置值.
在看一个复选框的例子,见[demo](http://jsfiddle.net/84xp4jan/18/),

```html
{{#list}}
  <li><input type='checkbox' name='{{selected}}' value='{{.}}'> {{.}}</li>
 {{/list}}
 
```
 在控制台执行```listview.get('selected')```,你会发现返回的是一个数组,包含已选择的checkbox的值.
 最后再看一个```select```的[demo](http://jsfiddle.net/84xp4jan/19/)吧.
 
 通常双向绑定都会提供一个watch功能,在ractivejs中实现watch是```observe```方法,它监听绑定数据的变化,在上面的radiobox的demo上我们扩展一个功能,当选中error时,弹出一个提示框,见[demo](http://jsfiddle.net/84xp4jan/20/),```observe```的代码如下:
 
```javascript
  listView.observe('status',function(newvalue,oldvalue){
    if(newvalue==="Error"){
        alert('你选择了Error')
    }
})

```
 observe的回调中返回newcalue与oldvalue,分别对应的是改变之后与改变之前的值.其它对绑定数据的操作例如push、splice都会触发```observe```,例如[demo](http://jsfiddle.net/84xp4jan/23/),但点击按钮时list数组就会添加一个元素,同时会执行observe.更多有关observe的用法请看对应的[文档](http://docs.ractivejs.org/latest/ractive-observe)
 
### 四.事件处理
 使用ractivejs时，我们使用代理事件的方法,首先在模板声明代理事件,然后在js代码中订阅事件,例如[demo](http://jsfiddle.net/84xp4jan/24/),
 
```html

{{#list:index}}
    <li> {{.}}<button on-click="del:{{index}}">删除</button></li>
 {{/list}}
  <p><input type="text" value="{{name}}" ><button on-click="add">添加</button></p>
  <p><button on-click="show">查看当前列表的值</button></p>
```

```javascript
listViewevent.on({
    add:function(){
        var name = this.get('name');
        if(name.replace(/(^\s*)|(\s*$)/g, "").length===0){
            return
        }
        this.push('list',name);
    },
    del:function(event,index){
        this.splice('list',index,1)
    }
})

```

特别注意的是上面的```del```方法,我们传了一个index参数，这个参数表示当前数组元素的索引,这个index值是在模板中通过```del:{{index}}```方式传进去的.同时我们可以看到,大部分逻辑都是在操作数据,ractivejs官方也提供了很多操作数据的[方法](http://docs.ractivejs.org/latest/get-started).

### 结尾
以上列举了ractivejs常用的一些特性,我做了一个稍微复杂的[demo](http://jsfiddle.net/84xp4jan/27/),这个例子包含了上面介绍的大部分特性.接下来会介绍怎样基于ractive将复杂业务拆分不同的功能模块.
