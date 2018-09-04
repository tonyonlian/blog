---
title: 验证模块joi介绍
date: 2017-08-18 22:31:00
tags: node 
---
&emsp;&emsp;joi是nodej的一个工具模块，主要用于JavaScript对象的校验。它是一种简单易用的javacript对象约束描述语言，可以轻松解决nodejs开发中的各种参数的校验。
<!--more-->
详细资料见 https://github.com/hapijs/joi/tree/v10.5.0
##### 1. nodejs引入： 
```
const Joi = require('joi');
```
##### 2.  开时使用：
```
const schema = Joi.object().keys({
    username: Joi.string().alphanum().min(3).max(30).required(),
    password: Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/),
    access_token: [Joi.string(), Joi.number()],
    birthyear: Joi.number().integer().min(1900).max(2013),
    email: Joi.string().email()
}).with('username', 'birthyear').without('password', 'access_token');


const result = Joi.validate({ username: 'abc', birthyear: 1994 }, schema);
//result --> { error: null, value: { username: 'abc', birthyear: 1994 } }
//result.error === null, 说明校验通过 ,result.value校验的对象

//也可以使用回调函数，异步获取校验结果，
Joi.validate({ username: 'abc', birthyear: 1994 }, schema, function (err, value) { 

});
```
上面的例子定义了一个schema，包含的意思如下：
- username 
               
> Joi.String() -- 定义类型必须是字符串类型

>.alphanum() -- 定义必须包含字母或数字

>.min(3).max(30) -- 定义字段长度3-30

>.required() -- 定义必修字段

>.with('username','birthyear') -- 校验对象字段必须和birthyear同时存在

- password

>.regex(/^[a-zA-Z0-9{3-30}$/) -- 定义字段必须匹配正则规则。

>.without('password','access_token') -- 校验对象中，'password'与'access_token'不同是存在

- access_token
>.[Joi.string(),Joi.number()] -- 定义字段类型为数字类型或字符串类型

>没有.required()约束 -- 定义字段为可选字段

- birthyear

>.Joi.number().integer() -- 定义字段为数字整型

>.min(1900).max(1994) -- 定义字段值范围在1900-1994

- email

>.email() -- 定义字段为邮箱地址

##### 23.  用法：
 使用分两步完成 第一步使用joi提供的类型与约束定义一个schema 
 ```
 const schema = {
    a: Joi.string()
};
 ```
 &emsp;&emsp;注意：joi schema对象是不可变，这意味着每增加一条规则（e.g. .min(5)）将会返回一个新的schema对象。

第二步 通过定义的schema完成值的校验
```
const {error, value} = Joi.validate({ a: 'a string' }, schema);

// or

Joi.validate({ a: 'a string' }, schema, function (err, value) { });
```
如果输入的值校验有效，则err===null 否则将会返回错误对象
schema可以使用普通的JavaScript对象，其中对象的字段关联joi类型来定义。也可以直接使用一个joi类型定义

```
const schema = Joi.string().min(10);
```
如果schema是直接使用joi类型定义 ，则可以直接使用schema.validate(value,callback)验证。如果是一个非joi类型对象的定义的shema ，实质joi模块会转换成object() 类型，等同如下：
```
const schema = Joi.object().keys({
    a: Joi.string()
});
```

joi模块的字符串编码默认为utf-8

 
 