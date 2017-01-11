---
layout: post
title:  "跨域访问"
date:   2016-08-09 13:30:09
categories: tech
---

# HTTP 访问控制

在实际业务中遇到这样一个问题,H5页面(`http://h5.ddd.com`)要调用Api服务器(`http://api.ccc.com`)的接口。根据浏览器同源策略,web应用程序通过`XMLHttpRequest `对象或`Fetch`能且只能向同域的资源发起HTTP请求,而不能向任何其它域名发起请求。这里通过一个例子实现说明怎样实现跨域请求(不是通过jsonp)

本地起两个http服务,端口要不同,模拟跨域

浏览器端：

``` javascript
  this.$http.get('http://localhost:7001/api/ding/test')
    .then((res)=>{
      console.log(res)
    })
    
```

//server端

``` javascript
var router = new ( require('koa-router') )();

//路由
router.get('/api/ding/ding/test',controller.mytest);

//controller
mytest:function *() {
        this.body = {
            result:'0'
        }
    }

```

刷新浏览器你可以看到浏览器控制台报错:

``` javascript
XMLHttpRequest cannot load http://localhost:7001/api/ding/test. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8083' is therefore not allowed access.
```
在服务端打断点可以看到请求可以接收到:

``` javascript
{ request: 
   { method: 'GET',
     url: '/api/ding/test',
     header: 
      { host: 'localhost:7001',
        connection: 'keep-alive',
        accept: 'application/json, text/plain, */*',
        origin: 'http://localhost:8083',
        'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36',
        referer: 'http://localhost:8083/',
        'accept-encoding': 'gzip, deflate, sdch, br',
        'accept-language': 'zh-CN,zh;q=0.8,en;q=0.6',
        cookie: 'Webstorm-ed8315e8=a17e1503-470b-4635-afb6-c9c081f3c3b5; _nav_status=0; JSESSIONID=61FF401A620EAC7E0B31E1FE48C39E36' } },
  response: { status: 404, message: 'Not Found', header: {} },
  app: { subdomainOffset: 2, proxy: false, env: 'DEV' },
  originalUrl: '/api/ding/test',
  req: '<original node req>',
  res: '<original node res>',
  socket: '<original node socket>' }
```
也就是说`跨域并非浏览器限制了发起跨站请求，而是跨站请求可以正常发起，但是返回结果被浏览器拦截了`。最好的例子是CSRF跨站攻击原理，请求是发送到了后端服务器无论是否跨域！注意：有些浏览器不允许从HTTPS的域跨域访问HTTP，比如Chrome和Firefox，这些浏览器在请求还未发出的时候就会拦截请求，这是一个特例。

要实现跨域,我们可以在http相应头中设置`Access-Control-Allow-Origin`,

``` javascript

var allowOriginUrl = 'http://localhost:8083';
var requestHeader = this.request.header;

if( requestHeader.origin &&requestHeader.origin.indexOf(allowOriginUrl) >-1 ){

        this.set("Access-Control-Allow-Origin", requestHeader.origin);
}

```
这样通过就可以跨域访问资源了.





