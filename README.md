# cookie-session-2-
node中cookie和session的区别及使用(2)[转]

首先，我必须义正言辞的吐槽一下这个宇宙级框架！express3.x和expss4.x差别怎么就那么大呢？找了好多资料来学习，但总是莫名其妙的报错，一开始我以为是因为我长得不好看，后来发现。。。我用的是4.x的express，而教程是3.x的，好多都对不上号。我@#￥%……&*（）&……￥

好了，吐槽结束，进入正题。作为一个励志改变世界的程序员，我们必须紧跟时代的潮流，所以nodejs死亡笔记都是基于express4.x的。本篇文章将讲解cookie和
session。

我之前写过一篇express项目搭建的博客，所以如何搭建我就不说了。

## cookie

在web应用中，多个请求之间共享“用户会话”是非常必要的。但HTTP1.0协议是无状态的。那这时Cookie就出现了。那Cookie又是如何处理的呢？

Cookie的处理：
```
服务端向客户端发送Cookie
客户端的浏览器把Cookie保存
然后在每次请求浏览器都会将Cookie发送到服务端
在HTML文档被发送之前，Web服务器通过传送HTTP 包头中的Set-Cookie 消息把一个cookie 发送到用户的浏览器中，如下示例：

Set-Cookie: name=value; Path=/; expires=Wednesday, 09-Nov-99 23:12:40 GMT;
```
其中比较重要的属性：
```
name=value：键值对，可以设置要保存的 Key/Value，注意这里的 name 不能和其他属性项的名字一样
Expires： 过期时间（秒），在设置的某个时间点后该 Cookie 就会失效，如 expires=Wednesday, 09-Nov-99 23:12:40 GMT
maxAge： 最大失效时间（毫秒），设置在多少后失效
secure： 当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效
Path： 表示 cookie 影响到的路，如 path=/。如果路径不能匹配时，浏览器则不发送这个Cookie
```
### cookie在express中的使用

express 在 4.x 版本之后，管理session和cookies等许多模块都不再直接包含在express中， 而是需要单独下载安装相应模块。

cookieParser安装： 
$ npm install cookie-parser

通过express命令创建的项目会自动将这个模块安装。
使用方法：
```
app.get('/', function (req, res) {
    // 如果请求中的 cookie 存在 isVisit, 则输出 cookie
    // 否则，设置 cookie 字段 isVisit, 并设置过期时间为1分钟
    if (req.cookies.isVisit) {
        console.log(req.cookies);
        res.send("再次欢迎访问");
    } else {
        res.cookie('isVisit', 1, {maxAge: 60 * 1000});
        res.send("欢迎第一次访问");
    }
});

```
## session

### 什么是session？

session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而session保存在服务器上。

客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上，这就是session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。

如果说Cookie机制是通过检查客户身上的“通行证”来确定客户身份的话，那么session机制就是通过检查服务器上的“客户明细表”来确认客户身份。

session相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。

两者的区别：
```
cookie数据存放在客户的浏览器上，session数据放在服务器上。

cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗 考虑到安全应当使用session。

session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能 考虑到减轻服务器性能方面，应当使用COOKIE。

单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

所以建议：将登陆信息等重要信息存放为session、其他信息如果需要保留，可以放在cookie中
```
### express中的session

跟cookie一样都需要单独的安装和引用模块， 安装模块：

$npm install express-session
这个模块没有默认安装，需要自己手动安装，并引入： 
var session = require('express-session');
这个模块的主要的方法是 session(options)，其中 options 中包含可选参数，主要有：
```
name: 设置 cookie 中，保存 session 的字段名称，默认为 connect.sid 。
store: session 的存储方式，默认存放在内存中，也可以使用 redis，mongodb 等。express 
生态中都有相应模块的支持。
secret: 通过设置的 secret 字符串，来计算 hash 值并放在 cookie 中，使产生的 signedCookie 
防篡改。
cookie: 设置存放 session id 的 cookie 的相关选项，默认为 (default: { path: ‘/’, 
httpOnly: true, secure: false, maxAge: null })
genid: 产生一个新的 session_id 时，所使用的函数， 默认使用 uid2 这个 npm 包。
rolling: 每个请求都重新设置一个 cookie，默认为 false。
resave: 即使 session 没有被修改，也保存 session 值，默认为 true。
```
使用方法：
```
app.use(session({
    secret: 'hubwiz app', //secret的值建议使用随机字符串
    cookie: {maxAge: 60 * 1000 * 30} // 过期时间（毫秒）
}));
app.get('/session', function (req, res) {
    if (req.session.sign) {//检查用户是否已经登录
        console.log(req.session);//打印session的值
        res.send('welecome <strong>' + req.session.name + '</strong>, 欢迎你再次登录');
    } else {
        req.session.sign = true;
        req.session.name = 'https://github.com/CleverFan';
        res.send('欢迎登陆！');
    }
});

```
### session的持久化

重新运行npm start，然后刷新访问测试页面。我们会发现session丢了！ 这是因为session会默认的存储到内存当中。也就是说session数据都是存储在内存当中的，当进程退出后，session数据就会丢失。

在开发环境中，这也许并不算什么坏事。但是如果线上的应用是这样的，用户绝对是不能忍受的。所以，我们需要将session数据 持久化存储。

首先我们讲解如何把session存储到mongodb数据库当中：

在使用mongodb存储时首先要加载一个模块：connect-mongo以及mongoose

安装命令： 
```
npm install connect-mongo 
npm install mongoose

var mongoose = require("mongoose");
var session = require('express-session');
var MongoStore = require('connect-mongo/es5')(session);

mongoose.connect('mongodb://127.0.0.1:27017/sessiontest'); //连接数据库
mongoose.connection.on('open', function () {
    console.log('-----------数据库连接成功！------------');
});

app.use(session({
    secret: "what do you want to do?", //secret的值建议使用随机字符串
    cookie: {maxAge: 60 * 1000 * 60 * 24 * 14}, //过期时间
    resave: true, // 即使 session 没有被修改，也保存 session 值，默认为 true
    saveUninitialized: true,
    store: new MongoStore({
        mongooseConnection: mongoose.connection //使用已有的数据库连接
    })
}));
```
mongodb我就不讲了，不是本文重点，需要创建sessiontest数据库。

Redis是一个非常适合用于Session管理的数据库。

它的结构简单，key-value的形式非常符合SessionID-UserID的存储；
读写速度非常快；
自身支持数据自动过期和清除；
语法、部署非常简单。
基于以上原因，很多Session管理都是基于Redis实现的。所以我们这个示例将用redis管理session。

Express已经将Session管理的整个实现过程简化到仅仅几行代码的配置的地步了，你完全不用理解整个session产生、存储、返回、过期、再颁发的结构，使用Express和Redis实现Session管理，只要两个中间件就足够了：
```
express-session
connect-redis
```
参数
```
client 你可以复用现有的redis客户端对象， 由 redis.createClient() 创建
host Redis服务器名
port Redis服务器端口
socket Redis服务器的unix_socket
```
可选参数
```
ttl Redis session TTL 过期时间 （秒）
disableTTL 禁用设置的 TTL
db 使用第几个数据库
pass Redis数据库的密码
prefix 数据表前辍即schema, 默认为 “sess:”
```
如何使用：
```
var express = require('express');
var session = require('express-session');
var RedisStore = require('connect-redis')(session);

var app = express();
var options = {
    "host": "127.0.0.1",
    "port": "6379",
    "ttl": 60 * 60 * 24 * 30,   //session的有效期为30天(秒)
};

// 此时req对象还没有session这个属性
app.use(session({
    store: new RedisStore(options),
    secret: 'express is powerful'
}));
```
没有实际应用的博客不是好博客

最后，用一个简单的登录验证小项目结束本篇文章。

index.ejs:
```
<form action="/sign" method="post">
    <fieldset>
        <legend>Please sign in</legend>
        <p>User: <input type="text" name="user"/></p>

        <p>Pass: <input type="text" name="password"/></p>
        <button>Submit</button>
    </fieldset>
</form>

```
sign.ejs
```
<!doctype html>
<html>
<head>
    <title>session</title>
</head>
<body>
   <!--登录成功展示的内容-->
   <p>welecome <strong> <%=session.name%> </strong>, 欢迎你再次登录<a href="/out"></a></p>
</body>
</html>
```
user.json
```
{
    "admin":{
        "password": "admin",
        "name": "demo"
    }
}
```
admin为登录用户名，password为登录密码，name为用户昵称。
app.js
```
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var  session = require('express-session');

var routes = require('./routes/index');
var user = require('./user.json');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

// app.use('/', routes);
// app.use('/users', users);
app.use(session({
  secret: 'secret', //为了安全性的考虑设置secret属性
  cookie: {maxAge: 60 * 1000 * 30}, //设置过期时间
  resave: true, // 即使 session 没有被修改，也保存 session 值，默认为 true
  saveUninitialized: false, //
}));

app.get('/', function (req, res) {
  if (req.session.sign) {//检查用户是否已经登录，如果已登录展现的页面
    console.log(req.session);//打印session的值
    res.render('sign.ejs', {session:req.session});
  } else {//否则展示index页面
    res.render('index.ejs', {title: 'index'});
  }
});

//登录表单处理
app.post('/sign', function (req, res) {
  //登录的数据和user.json中的数据进行对比
  if(!user[req.body.user]){
    res.end("input wrong");
  }
  if (req.body.password != user[req.body.user].password || !user[req.body.user]) {
    res.end('sign failure');
  } else {
    req.session.sign = true;
    req.session.name = user[req.body.user].name;
    res.send('welecome <strong>' + req.session.name + '</strong>，<a href="/out">登出</a>');
  }
});

app.get('/out', function(req, res){
  req.session.destroy();
  res.redirect('/');
});


// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handlers

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
      message: err.message,
      error: err
    });
  });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}
  });
});


module.exports = app;

```
现在启动项目，就可以使用这个demo了。

项目源代码我会放在github上，感兴趣的可以去下载。 
github地址：https://github.com/CleverFan/nodejsStudy/tree/master/expressTest/day02_session
