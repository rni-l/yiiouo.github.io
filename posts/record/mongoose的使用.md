---
title: mongoose的使用
date: 2017-08-10 11:14:37
tags: ["nodejs", "mongodb"]
categories: ["记录"]
draft: true
---

> 学习了mongodb后，接着就是和nodejs进行对接，怎么在nodejs上使用mongodb。我这里选择的是mongoose，至于为啥，可能教程稍微多点吧。不过，这个mongoose还是有点复杂的。。。

[官方文档](http://mongoosejs.com/docs/guide.html)

[中文文档](https://mongoose.shujuwajue.com/guide/schemas.html)

# 使用

    // 安装包
    npm i mongoose -S

我nodejs使用的框架是`express`。

这是 4.x.x 版本

    // 首先导入mongoose
    var mongoose = require('mongoose')
    // 实例化一个骨架
    var Schema = mongoose.Schema
    /*
      Schema里面的参数，{ paw: String }
      是规定，执行操作的时，约定的字段和类型
      让格式更清晰吧
      而且可以定义默认值
    */
    var testSchema = new Schema({
      name: { type: String, default: 'Val'},
      paw: String
    })
    
    /* 
      为"users"的model，添加自定义方法
      这个"users"是你要操作的集合（collection）
    */
    testSchema.methods.findSimilarTypes = function(cb) {
      // .find({})这些就跟操作mongodb时一样了，cb就是回调函数
      return this.model('users').find({ name: this.name }, cb)
    }
    testSchema.methods.findSimilarTypes2 = function(cb) {
      return this.model('users').find({ paw: this.paw }, cb)
    }
    
    // 将骨架发布为model，同理这个"users"是要操作的集合
    var Test = mongoose.model('users', testSchema)
    
    // 实例一个实体(entity)，传一个{}参数
    var _entity = new Test({ name: '232', paw: '22' })
    
    // 执行之前定义的方法，打印获取到的数据
    _entity.findSimilarTypes2(function(err, data) {
      console.log(data)
    })

5.x.x 版本

```javascript
const mongoose = require('mongoose')

const db = mongoose.connect('mongodb://localhost/koa')

const Schema = mongoose.Schema,
  ObjectId = Schema.ObjectId

mongoose.Promise = global.Promise

db.then(async (result) => {
  console.log('连接 mongodb 成功')
  const News = new Schema({
    newsId: ObjectId,
    title: Date,
    body: String,
    date: {
      type: Date,
      default: new Date()
    }
  })

  const Model = mongoose.model('NewsModel', News)
  const child = new Model()
  child.title = '132sdfs'
  child.body = 'test body'
  const data = await child.save().catch(error => {
    console.log(error)
  })
  Model.find({}, (error, data) => {
    if (error) {
      return console.error('mongodb error:', error)
    }
    console.log('find data:', data)
  })
}).catch(error => {
  console.error.bind(console, '连接错误：')
})

```

5.x.x 版本， mongoose.connect() 返回的不再是一个普通对象，而是一个 Promise 对象。用 then 和 catch 来判断连接成功、失败。

总结下：

1. 首先定义一个骨架，约定好字段和字段的类型(String, Array, Number这些，还有很多种)
2. 为这个骨架添加自定义方法，这里就是要对数据库CRUD了
3. 然后根据这个骨架，发布一个model，对要操作的数据库，定义好
4. 接着就实例一个实体（entity），传入之前约定好的字段
5. 最后就是执行之前自定义好的方法，获取到数据

还有，不一定使用自定义方法，mongoose内置很多定义的方法

下面我们详细说下每个东西的详细值

## Schema

Schema，就是这个操作的骨架，形式，约定好字段和字段的类型。例如在后面的操作，会出现什么字段，是什么类型的，一些操作，都在这里定义好。

### 参数

* String
* Number
* Date
* Buffer
* Boolean
* Mixed
* Objectid
* Array

    new Schema({
      name: String,
      data: Array,
      list: {
        type: Array,
        default: [1, 2, 3]
      }
      msg: {
        type: String,
        default: 'abc',
        lowercase: true, // 小写
        trim: true // 去空格
      },
      updated: { type: Date, default: Date.now },
      age: { type: Number, min: 18, max: 65 },
    })

可以看到，有很多内置方法，这些都能减轻工作

## model

### 内置方法

先定义个骨架
​    
    var User = new Schema({
      name: String,
      pwd: String,
      avatar: String,
      createdTime: {
        type: Date,
        default: Date.now
      },
      email: String,
      token: String
    })

再生成一个model

    var Test = mognoose.model('user', User)

使用内置的方法

    Test.find(function(error, data){
      console.log(data)
    })


