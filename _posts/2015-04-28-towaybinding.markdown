---
layout: post
title:  "前端开发中的双向绑定"
date:   2015-04-25 16:15:09
categories: tech
---
说到数据的双向绑定,做过.NET开发的同学应该很熟悉,我依稀记得在Datalist这个控件中可以绑定一个数据源然后repeater什么的,这样既可以读取数据源的值又可以修改数据源的值,可以很方便的进行数据的更新.那么在前端开发中数据的双向绑定是什么?简单的说就是数据对象发生变更以后要及时的更新DOM,当用户操作改变DOM树以后要更新数据对象.来看一个简单的例子,现有这样一个需求:

{% highlight javascript %}

<input type="text" value="" id="ipt"  />
<span id="sname"></span>

var personObj = {name:"jack"}
{% endhighlight %}

当personObj的name属性改变时,要即时更新input的value值与span的文本值,同样的当你改变界面input的值的时候personObj对象的name属性也要同步更新.相信大家通过jQuery操作DOM可以非常容易的实现这个功能.现在我们通过原生的javascript实现数据的双向绑定.
ES5新推出了一个类似存取器的特性[Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty),通过它可以很容易的实现数据绑定,[demo](http://jsfiddle.net/t7nj38L3/)

{% highlight javascript %}
var ipt = $("#ipt");
var sname = $("#sname");
Object.defineProperty(personObj,'name',{
	get:function(){
		return ipt.val()
	},
	set:function(val){
		ipt.val(val);
		sname.text(val);
	},
	enumerable:true,
	configurable:true
})

ipt.keyup(function(){
	personObj.name = $(this).val();
})

{% endhighlight %}

应用ES7的新特性[Object.observe](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/observe)同样可以实现数据的绑定

- 模型到视图的绑定:
 
{% highlight javascript %}
var personObj = {
	name:""
};
Object.observe(personObj,function(changes){
	changes.forEach(function(changes){
		ipt.val(personObj.name);
		sname.text(personObj.name)
	});
});
personObj.name = "jack";
{% endhighlight %}

- 从视图到模型的绑定

{% highlight javascript %}
ipt.keyup(function(){
	personObj.name = $(this).val();
})

{% endhighlight %}

现在能实现数据绑定的框架有很多,例如angualrjs、Knockout,angularjs实现上述功能更简单,通过```ng-model```直接就可实现:

{% highlight javascript %}
<div ng-controller="testCtrl">
    <input type="text" value="" ng-model="userName"  />
    <span ng-bind="userName"></span>
</div>
 
{% endhighlight %}

angualrjs的双向绑定具体体现在ng-model指令上,页面上的userName由```$watch```来监听,每当执行```keydown```事件时都会产生一个$digest循环,通过通过脏值检测来实现数据的双向绑定，我们可以用一段简洁的代码来模拟angular中实现双向绑定的$watch与$digest:

{% highlight javascript %}
 var Scope = function(){
        this.$$watchers = []
    };

    Scope.prototype.$watch = function( watchExp, listener ) {
        this.$$watchers.push( {
            watchExp: watchExp,
            listener: listener || function() {}
        } );
    };

    Scope.prototype.$digest = function(){
        var dirty;
        do {
                dirty = false;
                for( var i = 0; i < this.$$watchers.length; i++ ) {
                    var newValue = this.$$watchers[i].watchExp(),
                        oldValue = this.$$watchers[i].last;

                    if( oldValue !== newValue ) {
                        this.$$watchers[i].listener(newValue, oldValue);

                        dirty = true;

                        this.$$watchers[i].last = newValue;
                    }
                }
        } while(dirty);
    };
    
{% endhighlight %}

具体的实例请看[这里](http://jsfiddle.net/83L5u62v/).通常网页上界面刷新操作都会对应一个具体的事件,比如click事件会使界面刷新、AJAX请求等,angular封装了一些常用的操作函数例如ng-click,$timeout,$http等,这些操作完成时都会调用```$digest```.在angular中并不推荐直接调用```$digest```,而是封装在```$apply```中,
angular源码:

{% highlight javascript %}
$apply: function(expr) {
        try {
          beginPhase('$apply');
          return this.$eval(expr);
        } catch (e) {
          $exceptionHandler(e);
        } finally {
          clearPhase();
          try {
            $rootScope.$digest();
          } catch (e) {
            $exceptionHandler(e);
            throw e;
          }
        }
      }
 {% endhighlight %}           

apply方法其实直接调用了digest方法,增加这个校验主要是位了对expr进行校验,如果不通过就抛除异常.现在看下$http的源码:

{% highlight javascript %}
if (useApplyAsync) {
          $rootScope.$applyAsync(resolveHttpPromise);
  } else {
          resolveHttpPromise();
          if (!$rootScope.$$phase) $rootScope.$apply();
  }
  
{% endhighlight %}
在末尾出直接直接调用了$apply()!

新版的Alimonitor因为业务的关系采用了```Ractive```这个小而美的JS库,它专注于数据绑定、数据处理等多个功能.Ractive通过模板机制构建了一个virtualDOM,虚拟DOM的模板部分是通过他们所包含的```keypaths```注册到对应的viewmodel上,具体的双向绑定的细节还没有仔细的研究,下次再说。
