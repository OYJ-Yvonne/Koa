![](/markdown/Koa-share/asset/1.png)

>ouyangjing

----
##Introduction
* koa是基于node平台的下一代web开发框架。
* 继承了ES6的新属性，使用了generator和异步编程，可以免除重复繁琐的回调函数嵌套，让原来的异步回调变得简单，提升了错误处理的效率。
* 只提供了一个简单的server，具有更轻量级，更强的扩展性的优点。

----

##koa compared with Express
* Express主要基于Connect中间件框架，功能丰富，随取随用，并且框架自身封装了大量便利的功能，比如路由、视图处理等等。
* koa框架自身并没集成太多功能(模块)，大部分功能需要用户自行require中间件去解决，由于其基于ES6 generator特性的中间件机制，让异步逻辑用同步写法实现。

----
##Install and Perform
<p style="text-align:left">1、install</p>

* koa v1 需要 >0.11.x的 node版本，koa v2需要 >7.6的node版本。
```
> npm install -g koa-generator
> koa new-project && cd new-project
> npm install
```
----
<p style="text-align:left">2、manually install</p>

* 创建一个目录hello-koa、package.json、app.js：
<img src="/markdown/Koa-share/asset/pakage-json.png" alt="" style="width: 40%;height: 40%;">

* 然后在hello-koa目录下执行命令：
```
C:\...\hello-koa> npm install 
```
----
##Perform
* 为了完全支持ES6的语法，在和谐模式下加载koa源代码 。 
```
node --harmony app.js
```
或者
```
npm start
```
```
"scripts": {
    "start": "node app.js"
}
```
----
##The construction of the project
![](/markdown/Koa-share/asset/2.png)

----
##Basic Usage
* Koa 应用是一个包含一系列中间件函数的对象。 
* 这些中间件函数基于 request 请求以一个类似于栈的结构组成并依次执行。 
* 核心设计思路是为中间件层提供高级语法糖封装，以增强其互用性和健壮性。

----
##1、app.listen(...)
* 如下为一个绑定3000端口的简单 Koa 应用：
```
var koa = require('koa');
var app = koa();
app.listen(3000);
```
app.listen(...) 实际上是以下代码的语法糖:
```
var http = require('http');
var koa = require('koa');
var app = koa();
http.createServer(app.callback()).listen(3000);
```
```
var https = require('https');
https.createServer(app.callback()).listen(3000);
```
* koa可以同时支持 HTTP和 HTTPS，且能在多个端口监听同一个应用。
```
For example （hello-world.js）
```
----
##2、context
* Koa 提供一个 Context 对象，表示一次对话的上下文（包括 HTTP 请求和 HTTP 回复）。通过加工这个对象，就可以控制返回给用户的内容。
```
For example （context.js）
```
----
##3、app.keys
* 设置Cookie的密钥，该密钥会被传递给 KeyGrip。
```
app.keys = ['im a newer secret', 'i like turtle'];
KeyGrip（keys，algorithm，encoding）
```
也可以自己生成 KeyGrip 实例：
```
app.keys = new KeyGrip(['im a newer secret', 'i like turtle'], 'sha256');
```
* 在进行cookie签名时，只有设置 signed 为 true 的时候，才会使用密钥进行加密：
```
this.cookies.set('name', 'tobi', { signed: true });
```
```
For example（keyGrip.js）
```
----
##4、HTTP Response的类型
* default type : text/plain
* ctx.request.accepts判断请求类型，客户端希望接受什么数据，然后使用ctx.response.type指定返回类型。
```
For example (response.js)
```
----
##Route
####The original route
* 通过ctx.request.path获取用户请求的路径，由此实现简单的路由：
```
For example (route.js)
```

####Koa-route
* 原生路由用起来不太方便，我们可以使用封装好的koa-route模块：
```
For example (koa-route.js)
```
----
####Koa-static
* 一个网站肯定需要提供静态资源（图片、字体、样式表、脚本......），为它们一个个写路由就很麻烦，koa-static模块封装了这部分的请求。
```
For example (koa-static.js)
```

####Redirect
* 有些场合，服务器需要重定向（redirect）访问请求。比如，用户登陆以后，将他重定向到登陆前的页面。ctx.response.redirect()方法可以发出一个302跳转，将用户导向另一个路由。
```
For example (redirect.js)
```
----
##Middleware of koa
####1、concept
* 基本上，Koa 所有的功能都是通过中间件实现的，每个中间件默认接受两个参数，第一个参数是 Context 对象，第二个参数是next函数。只要调用next函数，就可以把执行权转交给下一个中间件。
```
For example (middleware.js)
```

####2、middle stack
* 多个中间件会形成一个栈结构，以"先进后出"的顺序执行。
```
For example (middle-stack.js)
```
----
* koa中间件的执行顺序如图所示：(从第一个中间件开始执行，遇到next进入下一个中间件，一直执行到最后一个中间件，在逆序，执行上一个中间件next之后的代码，一直到第一个中间件执行结束才发出响应)

<img src="/markdown/Koa-share/asset/3.jpg" alt="" style="width: 40%;height: 40%;">  
<img src="/markdown/Koa-share/asset/4.jpg" alt="" style="width: 40%;height: 40%;">

----
####3、异步中间件
* 上面所有例子的中间件都是同步的，不包含异步操作。如果有异步操作，中间件就必须写成 async 函数。
```
For example （async.js和async-await.js)
```
* 检测用户权限的middleware可以决定是否继续处理请求，还是直接返回403错误：
```
app.use(async (ctx, next) => {
    if (await checkUserPermission(ctx)) {
        await next();
    } else {
        ctx.response.status = 403;
    }
});
```
----
##Send json between sever and client
* 客户端：
```
var datas = {
　　foo:1,
　　bar:2
}
var ret = await fetch('localhost:3000/api/', {
　　method: 'POST',
　　headers:{
　　　　'Content-Type' : 'application/json'
　　},
　　body:JSON.stringify(datas)
});
```
* 服务端：
ctx.req.body或ctx.request.body拿不到响应的数据。
```
ctx.req.on('data', function(data){
　　console.log(data);
})
```
```
<Buffer 7b 22 66 6f 22 3a 31 2c 22 62 61 72 22 3a 32 7d>
```
----

##koa-body
```
import koabody from 'koa-body';
.....

app.use(koabody({}));

app.use(async ctx=>{
　　console.log(ctx.request.body);

　　ctx.body = {
　　　　message:"hello!"
　　}
})

```
----
##Error
####1、500错误
* Koa 提供了ctx.throw()方法，用来抛出错误，ctx.throw(500)就是抛出500错误。
```
const main = ctx => {
  ctx.throw(500);
};
```
运行后会看到一个500错误页"Internal Server Error"。

----
####2、处理错误的中间件
* 为了方便处理错误，最好使用try...catch将其捕获。但是，为每个中间件都写try...catch太麻烦，我们可以让最外层的中间件，负责所有中间件的错误处理。
```
For example (try-catch.js)
```
----
####3、释放error事件
* 如果错误被try...catch捕获，就不会触发error事件。调用ctx.app.emit()，手动释放error事件，才能让监听函数生效。
```
For example (emit.js)
```
----

##koa v1 and koa v2
* koa1.x的特点就是使用genetator来控制项目的同步，而koa2.x最大的特点就是使用async和await

>generator的写法：
```
app.use(function *(next){
  var start = new Date;
  yield next;
  var ms = new Date - start;
  this.set('X-Response-Time', ms + 'ms');
});
```
>await的写法
```
app.use(async (next) => {
  var start = new Date;
  await next();
  var ms = new Date - start;
  this.set('X-Response-Time', ms + 'ms');
});
```
----
* koa v2兼容koa v1的语法，但需要将基于generator的中间件用koa-convert转换一下才能使用。
```
const convert = require ( 'koa-convert' );
app.use( convert ( bodyparser ) );
app.use( convert ( json() ) );
app.use( convert ( logger() ) );
```
* 基于async/await实现中间体系的koa2框架将会是是node.js web开发方向大势所趋的普及框架。基于generator/yield的koa1将会逐渐被koa2替代，毕竟使用co.js来处理generator是一种过渡的方式，虽然有其特定的应用场景，但是用async/await会更加优雅地实现同步写法。
----
👉[koa](http://koa.bootcss.com/)

👉[koa2](https://chenshenhai.github.io/koa2-note/)

----
## That's all , Thanks!
