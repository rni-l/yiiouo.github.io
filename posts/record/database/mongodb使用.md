---
title: mongodb使用
date: 2017-08-04 15:35:12
tags: ["nodejs", "mongodb"]
categories: ["记录"]
---

> 记录了mongodb安装、配置和基本的使用

## 安装

1. 在官网下载你所需要的版本（我的是windows）
2. 安装（可以指定目录）
3. windows是要配置环境变量，不然会出现闪退的情况

在系统变量的PATH中，添加一个新的变量，是你mongodb下载的路径（我的是：C:\Program Files\MongoDB\Server\3.4\bin）

4. 可以选择你数据库文件放在的位置（好像默认是在C盘的）

在命令行，进入到mongod.exe的目录下，输入：

    mongod.exe --dbpath h:\mongodb\data\db

这样就能更改数据库数据存放的位置



### 4.x 版本在 linux(centos) 安装

[官方链接](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)

1. 在 `/etc/yum.repos.d/mongodb-org-4.4.repo` 创建安装资源的配置文件：

```shell
[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```

2. 安装依赖

`sudo yum install -y mongodb-org`

3. 修改日志，数据的存储位置

在 `/etc/mongod.conf` 进行修改

修改了目录，记得要设置文件夹的权限

4. 开启服务

`service mongod start`

### 报错

两种方式查看错误日志：

`systemctl status mongod.service` 和 去 mongodb 的日志文件查看

code === 100

应该是有文件没权限写入，解决方式：给对应的文件设置文件

```
# 具体设置权限，看你安排
sudo chmod -R 777 /db/mongo/data && sudo chown -R mongod /db/mongo/data && sudo chgrp  -R mongod /db/mongo/data
sudo chmod -R 777 /db/mongo/log && sudo chown -R mongod /db/mongo/log && sudo chgrp  -R mongod /db/mongo/log

# 复制原本的 mongodb 的文件夹的上下文到 当前的文件夹
chcon -R --reference=/var/lib/mongo /db/mongo/data
```



### 内存配置

[文档](https://docs.mongodb.com/manual/reference/parameters/#param.maxIndexBuildMemoryUsageMegabytes)



### 登录

如果没有配置账号信息： `mongod`

配置了：

`mongo -u “{用户名}" --authenticationDatabase “{数据库}" -p "{密码}”`




## 基础知识

### mongodb组成

1. database：数据库
2. collection：集合（表）
3. document：文档（row）
4. field：域（字段）
5. index：索引
6. primary key（主键，自动将_id设置为主键）

BSON是一种类json的一种二进制形式的存储格式,简称Binary JSON

mongodb可以说是json的格式

#### document

文档是一组**键值（Key-value）**对

##### 注意

1. 文档的键/值对是**有序**的
2. 不仅可以是在双引号里面的字符串，还可以是其他几种数据类型
3. 区分类型和大小写
4. 不能有重复的键
5. 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符

##### 命名规范

1. 键不能含有\0（空字符）
2. .和$有特别意义
3. 以'_'开头的键是保留（不推荐）

#### collection

##### 命名规范

1. 不能说空字符串
2. 不能含有\0（空字符）
3. 不能以'system.'开头

##### capped collections

固定大小的collection，性能更好

collection的数据存储空间值要提前分配

`db.createCollection("mycoll", {capped:true, size:100000})`

#### 元数据

数据库的信息存储在集合中

`dbname.system.*`

* .namespaces：列出所有名字空间
* .indexes：列出所有索引
* .profile：包含数据库概要信息
* .CN：列出所有可访问数据库的用户
* .sources：包含复制对端的服务器信息和状态

#### 数据类型

* String，在MongoDB中，UTF-8编码的字符串才是合法的
* Integer，整型，32位或64位
* Boolean
* Double，双精度浮点值
* Min/Max keys，将一个值与BSON元素的最低值和最高值相对比
* Arrays，数组或列表
* Timestamp，时间戳
* Object：内嵌文档
* Null
* Symbol，符号
* Date：日期时间
* Object ID，对象ID，用于创建文档的ID
* Binary Data：二进制数据
* Code：代码类型，用于在文档存储javascript代码
* Regular expression：正则

### 连接数据库

* 本地：`mongodb://localhost`
* 有用户名和密码：`mongodb://user:pwd@localhost`
* 数据库：`mongodb://fred:foobar@localhost/db`
* 连接服务器1和服务器2：`mongodb://example1.com:27017,example2.com:27017`


### 调试功能

`explain()` 

    db.CN.find().explain()

上面的代码，可以解析当前查询工作的详细情况

使用 `.explain( "executionStats" )` 可以查看操作耗费了多少时间



### 权限配置-添加管理用户

monogodb 默认不需要用户的，而且是没有用户的，超级管理员也没有，需要自己手动添加的。

首先，我们先在 mongodb 添加用户。比如，我现在要在 admin 数据库添加用户

    use admin
    db.createUser(
      {
        user: "admin",
        pwd: "123",
        roles: [ { 
          role: "readWrite",
          db: "admin" // 添加到 admin db里
        } ]
      }
    )

这样们就在 admin 库里添加了一个用户，可以使用 `db.system.users.find()` 查看当前库的用户

但是这个用户在其他库是用不了的，比如我想在 test 这个数据库登录用户： `db.auth('admin', '123')` ，会登录不了的，我们要在 test 再添加一个新的用户才可以

其他一些用户权限：

角色  | 介绍 
----- | -----
 read | 只读
 readWrite | 读写
 dbAdmin | 提供执行管理任务的能力，如模式相关任务，索引，收集统计信息。此角色不授予用户和角色管理权限
 dbOwner | 提供对数据库执行任何管理操作的功能
 userAdmin | 创建修改角色和用户功能

还有很多，详细看[官方文档](https://docs.mongodb.com/manual/core/security-built-in-roles/)

然后就是修改配置文件，这里的教程是 centos 的

根目录，执行 `vim etc/mongod.conf`

添加或者修改以下这段代码

    security:
      authorization: enabled

然后重启 mongodb ， `service mongod restart` ，再次进入 `mongo` ，就需要 `db.auth()` 后，才能进行相应的操作了

然后再数据库连接上，也需要配上用户和密码

    mongoose.connect(`mongodb://user:pwd@localhost/test`, { useMongoClient: true })




## API

### 常用命令

* show dbs：显示有什么数据库存在
* show tables：显示集合
* db：显示当前的数据库或集合
* use db：连接特定的数据库
* db.stats()：查看当前数据库信息
* db.CN.stats()：查看当前集合的信息
* db.serverStatus()：查看服务器状态
* db.enableFreeMonitoring(): 开启免费的性能监控
* db.disableFreeMonitoring()：关闭
* db.getSiblingDB("admin").shutdownServer()：关闭服务

### 查看库、集合大小

```shell
show dbs --> 查看每个库的大小
# 切换了库后
> db.stats(); --> 查看该库一些详细信息
{
	"db" : "data-collection",
	"collections" : 11,
	"views" : 0,
	"objects" : 530906,
	"avgObjSize" : 16670.245619751895,
	"dataSize" : 8850333421,
	"storageSize" : 3212660736,
	"indexes" : 32,
	"indexSize" : 16756736,
	"totalSize" : 3229417472,
	"scaleFactor" : 1,
	"fsUsedSize" : 220007739392,
	"fsTotalSize" : 250685575168,
	"ok" : 1
}
# 查看某个集合的信息
> db.behaviorerrors_copy1.stats();
{
	"ns" : "data-collection.behaviorerrors_copy1",
	"size" : 4236508972,
	"count" : 10582,
	"avgObjSize" : 400350,
	"storageSize" : 1294323712,
	"freeStorageSize" : 16384,
	"capped" : false,
	"wiredTiger" : {},
	"nindexes" : 4,
	"indexBuilds" : [ ],
	"totalIndexSize" : 561152,
	"totalSize" : 1294884864,
	"indexSizes" : {
		"_id_" : 200704,
		"applicationCode_1" : 86016,
		"createdAt_1" : 188416,
		"moduleCode_1" : 86016
	},
	"scaleFactor" : 1,
	"ok" : 1
}
```



### 删除数据库

进入目标数据库后，执行：`db.dropDatabase()`

### 删除collection

`db.CN.drop()`

### 查询耗时

一些方法可以使用 `.explain()` 知道该次查询的一些信息

```shell
db.numbers.explain().find({ num: { "$gt": 199992 } })
```

`.explain()` 默认返回这次查询计划信息，传入 `executionStats` 字段，会返回查询的信息；`allPlansExecution` 会返回所有信息



### 其他的一些方法

#### limit

限制读取数据的数量

    // 限制读取的数量为2
    db.CN.find({},{"title":1,_id:0}).limit(2)

#### skip

从第几个开始

    // 获取匹配到的数据，从第10行开始显示
    db.CN.find().skip(10)
    
    //混合使用，获取匹配到的数据，从10开始获取，获取100个
    db.CN.find().skip(10).limit(100)

#### sort

排序，1和-1，1为升序，-1为降序。

sort() => skip() => limit()

    // 以key值，升序排序
    db.CN.find().sort({KEY:1})



## Aggregation - 聚合

> Aggregation operations process data records and return computed results. Aggregation operations group values from multiple documents together, and can perform a variety of operations on the grouped data to return a single result

在用 mongodb 的时候，aggregation 会经常用到，它可以让你处理和重新组装数据，像常用的 `group`，可以用函数计算的 `accumulator`。可以将不同的操作组合在一起使用，比如 match, group, project, sort 等

```javascript
model.Behavior.aggregate([
  {
    $match: this.resetParams({
      status: 1
    })
  },
  {
    $group: {
      _id: '$name',
      list: {
        $push: '$$ROOT'
      }
    }
  }
])
```

上面这个 aggregation 操作，先通过 `$match` 获取指定数据，然后使用 `$group` 根据 `name` 字段进行分组，最后将分组的数据 `push` 到新的字段 `list` 里



## 索引

在 mongoose 配置：

```javascript
new Schema({
  name: {
    type: String,
    index: true,
    unique: true // 唯一索引
  }
})
```

查询索引值：`db.collection.getIndexes()`

删除索引：

db.collection.dropIndes(key)

db.collection.dropIndexes()



## mongose 常用方法

### 返回 id 而不是 _id

mongodb 返回的数据都会带有默认字段 `_id`，而我们想要的是 `id` 而不是 `_id` 的时候

```javascript
schema.virtual('id').get(function() {
  return this._id && this._id.toHexString ? this._id.toHexString() : this._id;
});
schema.set('toJSON', {
  virtuals: true,
  getters: true
});
```

通过 `mongose` 的 `schema` 内置的方法实现。使用 `schema.virtual` 定义一个虚拟字段即可，最后再设置 `schema.set(toJson, {.. })`

### 配置 createdAt 和 updatedAt

`mongose` 默认在这两个字段的配置，但用了 `monogose` 的配置，就不能在做一步格式化的操作，所以我这里做了一个封装

```typescript
// 配置公共 date 的配置
export const getDate = () => ({
  type: Number,
  default: Date.now,
  get: (v: number) => (v ? moment(v).format(FORMAT_DATE_TYPE) : v)
});

schema.pre('findOneAndUpdate', async function() {
  this.update(
    {},
    { $set: { updatedAt: Date.now() } }
  );
});
schema.pre('save', async function() {
  this.update(
    {},
    { $set: { updatedAt: Date.now() } }
  );
});
```

然后使用 `schema.pre` 的钩子，在有修改的操作的钩子里，对 `updatedAt` 进行更新



### 如何清空一个集合

```shell
# 删除该集合内的所有文档
db.behaviorerrors_copy1.remove({})

# 删除后，用 db.dd.stats() 查看信息
# 发下索引没清楚，totalSize 还是很大
db.behaviorerrors_copy1.stats();
{
	"ns" : "data-collection.behaviorerrors_copy1",
	"size" : 0,
	"count" : 0,
	"storageSize" : 336551936,
	"freeStorageSize" : 336543744,
	"totalIndexSize" : 561152,
	"totalSize" : 336551936,
	"indexSizes" : {
		"_id_" : 200704,
		"applicationCode_1" : 86016,
		"createdAt_1" : 188416,
		"moduleCode_1" : 86016
	},
}

# 删除集合的索引
# 删除后，还是有占用空间，但这是正常的，因为该集合有 4 个索引建，所有这些空间就是被这几个索引使用了
db.behaviorerrors_copy1.reIndex();
{
  "totalIndexSize" : 32768,
	"totalSize" : 45056,
	"indexSizes" : {
		"_id_" : 8192,
		"applicationCode_1" : 8192,
		"createdAt_1" : 8192,
		"moduleCode_1" : 8192
	},
}

```

简单来说要清空一个集合的方法有两个：

1. 直接 .drop 集合，然后再新建一个
2. .remove({}) + .reIndex() 进行清空操作



## 遇到的问题

### BSONObj 超出 16M 的限制

当某次查询，需要的内存超过 16M 的时候，mongodb 就会取消这次执行。特别是使用聚合时，很容易触发

暂时的解决方法：

1. 查询时，去除不必要的字段



### 要使用 useCreateIndex 配置

egg 配置：

```javascript
config.mongoose = {
    client: {
      url: 'mongodb://127.0.0.1/data-collection',
      // url: 'mongodb://10.10.5.64/data-collection',
      options: {
        useUnifiedTopology: true,
        useCreateIndex: true
      },
      // mongoose global plugins, expected a function or an array of function and options
    },
  };
```

要在初始化配置时进行配置

#### xxx.sock 文件 operation not permitted

删除它



