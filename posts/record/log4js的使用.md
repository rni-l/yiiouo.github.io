---
layout: nodejs
title: log4js的使用
date: 2017-09-20 14:55:44
tags: ["nodejs", "log4js"]
categories: ["记录"]
draft: true
---

# Nodejs log4js 的使用

>使用log4js，进行日志的管理，方便我们在生产环节进行调试和监控

## 如何使用

导入 `log4js` 包就行了

    npm i log4js -d
    // app.js
    var log4js = require('log4js')
    var logger = log4js.getLogger()
    logger.debug("Some debug messages")

上面的 `logger.debug()` 和 `console.log()` 使用的方式是一样的，当我们 `node app.js` 打开服务后，会看到有一条信息打印出来了

## Levels

总共有以下8个级别

* ALL
* FATAL
* ERROR
* WARN
* INFO
* DEBUG
* TRACE
* OFF

## Category

设置分类，日志可以进行分类，下面就是一个 test 分类的 logger

`var logger = log4js.getLogger('test')`

## Appender

打印日志的出口

    log4js.configure({
      appenders: [{
        type: 'dateFile', // 文件类型
        filename: 'logs/default.log', // 输出地址
        pattern: "-yyyy-MM-dd", // 文件输出模式
        alwaysIncludePattern: true // true的话，会模式生成一个新的日志文件
      }]
    })

## Filter

可以过滤掉你不想输出到文件的日志
    log4js.configure({
      level: 'DEBUG', // 低于 DEBUG 级别的日志不过输出到文件
      category: 'c1' // 只接受 c1 分类的日志，可接受数组
    })

## backups

设置备份数量

    {
      backups: 7
    }

只会保留7份日志文件

## 配置

    log4js.configure({
      level: 'INFO',
      appenders: [
        { type: 'console' }, // 控制台输出
        {
          type: 'dateFile', //文件输出
          filename: 'logs/app.log',
          pattern: "-yyyy-MM-dd",
          backups: 7,
          category: 'http'
        },
        {
          type: 'dateFile', //文件输出
          filename: 'logs/error.log',
          pattern: "-yyyy-MM-dd",
          backups: 7,
          category: 'error'
        }
      ]
    })
    // 打印相关的信息
    app.use(log4js.connectLogger(log4js.getLogger('http')))