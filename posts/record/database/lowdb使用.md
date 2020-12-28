---
title: lowdb使用
date: 2017-08-21 21:04:49
tags: ["nodejs", "lowdb"]
categories: ["记录"]
draft: true
---

# 使用

    cnpm i lowdb --save-dev


    // nodejs
    const low = require('lowdb')
    const FileSync = require('lowdb/adapters/FileSync')
    // 进行异步模式
    const adapter = new FileSync('db.json')
    // 实例一个db对象
    const db = low(adapter)
    // 设置默认值，并写入
    db.defaults({
      banner: [],
      category: [],
      product: [],
      usre: []
    }).write()

`db.get('banner[0]'), db.get('user.title'), db.get('banner[0].title')`，都是获取某个值，最后要加上`.value()`

`db.getState()`，查看db的状态

`db.setState({})`，清空db

`db.write().id`，自动插入随机id值

`db.has('posts').value()`，判断是否有这个值

`db.set('posts', []).write()`，将`posts`值设置为[]

使用`.push()`插入数据，在插入数据前，要先`get('filed')`，声明向谁插入，最后就写入`write()`

`db.get('banner').size().value()`，获取banner有多少条数据

`db.get('posts').find({ title: 'low!' }).assign({ title: 'hi!'}).write()`，获取`posts`字段，搜索`title`为'low!'的值，替换成'hi!'，最后写入

`db.get('posts').remove({ title: 'low!' }).write()` ，删除

`db.unset('user.name').write()`，删除一个字段


