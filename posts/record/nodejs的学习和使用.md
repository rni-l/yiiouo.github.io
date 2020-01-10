---
title: nodejs的学习和使用
date: 2017-08-04 16:52:41
tags: ["nodejs"]
categories: ["记录"]
draft: true
---

> 记录一些nodejs学习的内容

这篇文字，是(https://github.com/nswbmw/N-blog/)[https://github.com/nswbmw/N-blog/]这里进行学习nodejs，进行归纳的



## npm

### 常用命令

1. npm publish — 发包
2. npm unpublish \<packagename@version\> — 撤销包



1. [npm 教程](<https://juejin.im/post/5ab3f77df265da2392364341#heading-9>)

### 区别 dependencies、devDependencies

当在项目的根目录执行 `npm i`，dependencies 和 devDependencies 的依赖都会被下载

当该项目发布到 npm 上时，另外的项目 `npm i pro` ，安装你的项目时，只会安装 dependencies 的依赖包，devDependencies 不会被下载



### 版本控制

|     range     |                   含义                    |                       例                        |
| :-----------: | :---------------------------------------: | :---------------------------------------------: |
|    ^2.2.1     |   指定的 MAJOR 版本号下, 所有更新的版本   | 匹配 `2.2.3`, `2.3.0`; 不匹配 `1.0.3`, `3.0.1`  |
|    ~2.2.1     | 指定 MAJOR.MINOR 版本号下，所有更新的版本 | 匹配 `2.2.3`, `2.2.9` ; 不匹配 `2.3.0`, `2.4.5` |
|     >=2.1     |         版本号大于或等于 `2.1.0`          |               匹配 `2.1.2`, `3.1`               |
|     <=2.2     |          版本号小于或等于 `2.2`           |         匹配 `1.0.0`, `2.2.1`, `2.2.11`         |
| 1.0.0 - 2.0.0 |     版本号从 1.0.0 (含) 到 2.0.0 (含)     |         匹配 `1.0.0`, `1.3.4`, `2.0.0`          |
|               |                                           |                                                 |



## 使用express

cnpm i express --save

新建`index.js`文件

    var express = require('express')
    var app = express()
    
    app.get('/', function(req, res) {
      res.send('hello, 11')
    })
    
    app.listen(3000)

开启服务：`node indexjs`

## supervisor

安装在全局下：`npm install -g supervisor`

启动程序：`supervisor --harmony index`

这个包的作用，当你更改后，刷新网页会更新，而不用重新开启服务才刷新

