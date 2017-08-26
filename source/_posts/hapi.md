---
title: hapijs 框架的权限验证 
tags: node
---

###### hapijs 框架的权限验证demo，实现接口如下


```
/  get  登陆页面
/  post  提交登陆信息
/register  get  注册页面
/register  post  提交注册信息
/logout   get  退出接口

```
<!--more-->
###### 主要逻辑的实现

```javascript
//new hapi.server实例
var server = new Hapi.Server();

//设置服务器端口
server.connection({
    host: 'localhost',
    port: 7000
});

//注册相关插件

server.register([{
    register: CookieAuth // 权限插件
},
{
    register: Vision //视图模版插件
}


], function (err) {});

//设置视图
server.views({
    engines: {
        html: Handlebars
    },
    path: __dirname + '/views',
    layout: true
});

//权限设置及验证
    //验证函数
    var validation = function (request, session, callback) {
        console.log(session);
        var account = session.email;
        var user = User[account];
        if (!user) {
            return callback(null, false);
        }
        server.log('info', 'user authenticated');
        callback(err, true, user);

    }

    //session-cookie认证策略

    server.auth.strategy('session', 'cookie', true, {
        password: 'm!*"2/),p4:xDs%KEgVr7;e#85Ah^WYC',
        cookie: 'future-studio-hapi-tutorials-cookie-auth-example',
        isSecure: false,
        redirectTo: '/',
        validateFunc: validation
    });

//配置路由
  var routes = require('./router');
  server.route(routes);

//启动sever
 server.start(function (err) {
        if (err) {
            throw err
        }
        console.log('Server runing at:' + server.info.uri);
    });


```

###### 工程的文件说明

```
|-views 视图文件夹
    |-index.html 登陆页面
    |-profile.html 用户信息展示页面
    |-register.html 注册页面
|-router.js 路由文件
|-user_db.js 注册用户文件

```


##### demo 地址 ：https://github.com/tonyonlian/node-hapi-auth