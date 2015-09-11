---
layout: post
title:  "怎样基于ractivejs写components?"
date:   2015-09-10 16:17:09
categories: tech
---


![img](/assets/images/Component-Testing-cor1_2.png)

本文主要介绍怎样基于ractivejs写可复用的业务组件。ractivejs的组件都是基于`Ractive.extend()`方法来创建,首先简单的介绍下`Ractive.extend`方法的初始化配置.


## 初始化配置
组件的初始化配置与Ractive的[Initialisation Options](http://docs.ractivejs.org/latest/options)保持一致,举个例子,我们先初始化一个组件

```javascript
var textComponent = Ractive.extend({
  template: '<div>Hello,{{name}}!</div>'
});

```
然后在业务代码中实例化:[demo](http://jsfiddle.net/th2fywny/12/)

```javascript
new textComponent({
    el:"#text",
    data:function(){
        return{
            name:"小明"
        }
    }
});

```
或者在ractive中使用:[demo](http://jsfiddle.net/th2fywny/13/)

```javascript
var textView = new Ractive({
    el:"#text1",
    template:"<Component></Component>",
    data:function(){
        return{
            name:"alibaba"
        }
    },
    components: { Component: textComponent }
})
```
这里讲下配置中的`isolated`属性，isolated的意思是`隔离`,它的默认值是`false`,如果ractive对象中的component存在嵌套的情况,它是直接可以拿到外层ractive实例的`data`的;但如果isolated的值设置`true`,内部的component是接收不到data的值的,例如给上面`textComponent`加上`isolated:true`

```javascript
var textComponent = Ractive.extend({
  template: '<div>Hello,{{name}}!</div>',
  isolated:true
});

```
执行结果为:[demo](http://jsfiddle.net/th2fywny/14/)

```
Hello,!
```

## 数据绑定
在上面`textView`这个实例中`Component`需要的`name`值是在data中定义然后传递的,这里介绍一种新的数据绑定的方法,把上述代码改写下:[demo](http://jsfiddle.net/th2fywny/15/)

```javascript
var textComponent2 = Ractive.extend({
  template: '<div><input value={{val}} /></div>',
});

var textView2 = new Ractive({
    el:"#text1",
    template:"<div><Component val={{name}}></Component></div>",
    data:function(){
        return{
            name:"alibaba"
        }
    },
    components: { Component: textComponent }
})

```
`textComponent2`组件中`val`的值与`textView2`实例的`name`值绑定在一起,当name变化时val的值也会变化,反之亦然.这有点类似angular中directive的数据绑定.

## 组件的嵌套
日常项目中经常会遇到组件嵌套的情况,在ractive组件中如果遇到嵌套的情况则需要用到 `{{>content}}`或者`{{yield}}`,例如:[demo](http://jsfiddle.net/th2fywny/16/)

```javascript
var Outer = Ractive.extend({ 
    template: '<div>{{yield}}</div>'
});

var Inner = Ractive.extend({ 
    template: '<span>hello,world</span>' 
});

new Ractive({
    el: "#text2",
    template: '<outer><inner /></outer>',
    components: {
        outer: Outer,
        inner: Inner
    }
});

```
执行结果为: `<div><span>hello,world</span></div>`.将`Outer`中的`{{yield}}`换成`{{>content}}`,执行结果一样。但是二者还是有区别的.
例如:[demo](http://jsfiddle.net/th2fywny/16/)



```javascript
var Outer = Ractive.extend({ 
    template: '<div>{{yield}}</div>',
    oninit:function(){
    	this.on('inner.click',function(){
            console.log('inner')
        });
    }
});

var Inner = Ractive.extend({ 
    template: '<span on-click="click">hello,world</span>' 
});

var textView = new Ractive({
    el: "#text2",
    template: '<outer><inner /></outer>',
    components: {
        outer: Outer,
        inner: Inner
    }
});

```

在`outer`中给`inner`注册一个click事件,见[demo](http://jsfiddle.net/th2fywny/7/),可以看到点击事件并没有执行,这是因为通过`yield`引入的component并不属于外层的component(例如上面的Outer),它输入最外层的Ractive实例(例如textView对象),我们把oninit里面的注册事件加到textView中(见[demo](http://jsfiddle.net/th2fywny/8/)),click事件可执行.如果将上述代码中的 `yield` 改为 `>content`,


```javascript
var Outer = Ractive.extend({ 
    template: '<div>{{>content}}</div>',
    oninit:function(){
    	this.on('inner.click',function(){
            console.log('inner')
        });
    }
});

```
则在outer中为内部的component注册的事件可以执行,见[demo](http://jsfiddle.net/th2fywny/9/).

## 事件冒泡
跟DOM一样,如果组件嵌套则会出现事件冒泡的情况,例如[demo](http://jsfiddle.net/th2fywny/10/),点击inner在控制台可以看到会出现`inner,outer`，解决冒泡问题可以通过`return false`来解决,见[demo](http://jsfiddle.net/th2fywny/11/).

## Single-file components
[single-file](https://github.com/ractivejs/component-spec/)其实就是一个HTML文件,它包含了template、style、以及对应的组件逻辑,这种方式有点类似webComponent,例如写一个Grid组件,

```html

<table class="table">
<tbody>
    <tr>
        {{#columns}}
            <td>{{.label}}</td>
        {{/columns}}
    </tr>
    {{#rows:rowIndex}}
        <tr>
            {{#columns:columnIndex}}
                <td>
                    {{rows[rowIndex][.field]}}
                </td>
            {{/columns}}
        </tr>
    {{/rows}}
</tbody>
</table>
<style>

</style>
<script>
component.exports= {
	   onint:function(){
    },
        data: {
            columns: [],
            rows: []
    }
}
</script>

```

怎样去使用这个component?这里必须用到[ractive-load](https://github.com/ractivejs/ractive-load),因为ractive-load能将component文件转化为`Ractive.extend()`生成的对象.例如

```
<html>
<head><title>Ractive component demo</title></head>
<body>
  <main><!-- the component will be rendered here --></main>
  <script src='ractive.js'></script>
  <script src='ractive-load.js'></script>
  <script>
    var ractive;
    Ractive.load( 'Grid.html' ).then( function ( Grid ) {
      ractive = new Grid({
        el: 'main',
        data: {
        	columns: [
				{field: 'username', label: '用户名'},
				{field: 'name', label: '全称'},
				{field: 'email', label: '邮箱'}
			],
			rows: [
				{username: 'admin', name: 'Admin', email: 'admin@example.com'},
				{username: 'siwei', name: 'siwei', email: 'siwei@example.com'},
			]
         }
      });
    });
  </script>
</body>
</html>

```

你的应用里面如果用到的组件很少,用这种方式完全没有问题,但是对于大型的应用，用到的业务很多,这种方式就不可取了,因为这会造成大量的HTTP请求,所以这种sing-file类型的component应该要用对应的文件打包机制.

## 我们自己的解决方案
为了开发以及后期维护的便利,我们基于`sing-file`实现了一套业务组件开发机制-[RactiveComponent](https://github.com/blueforest/ractiveComponents),基本原理就是通过[rcu](https://github.com/ractivejs/rcu)和[ruc-builders](https://github.com/ractivejs/rcu-builders)将后缀为`.html`的组件文件转为CMD模块文件,最终将模块文件通过[browserify](http://browserify.org/)打包成一个`bundle.js文件`.描述这个过程的代码为:

```javascript
    var fs = require('fs');
    var rcu = require('rcu');
    var builders = require('rcu-builders');
    var Ractive = require('ractive');
    rcu.init( Ractive );

    grunt.registerTask('componentTojs',null,function(){
        fs.readdirSync( 'src/components' ).forEach( function ( htmlFile ) {
        var html = fs.readFileSync( 'src/components/' + htmlFile,'utf8').toString();

        var component = rcu.parse( html );

        // JavaScript module
        var commonJs = builders.cjs( component );
        fs.writeFileSync( 'src/compiled-components/' + htmlFile.replace( '.html', '.js' ),commonJs );
        });
    });


```
具体怎样使用可以将RactiveComponent仓库 clone到本地运行.

## 链接
- [ractive-component](http://docs.ractivejs.org/latest/components)
- [component-nesting](http://stackoverflow.com/questions/28681820/ractivejs-component-nesting)
- [component-spec](https://github.com/ractivejs/component-spec/)

