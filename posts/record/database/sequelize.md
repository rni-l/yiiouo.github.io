---
title: sequelize
date: 2020-10-10 15:07:00
tags: ["sequelize", "mysql"]
categories: ["记录"]
draft: true
---

> 以下使用，基于 EggJs 和 TypeScript

## Egg 配置

```javascript
config.sequelize = {
  dialect: 'mysql',
  host: '127.0.0.1',
  password: '123456',
  username: 'root',
  port: 3306,
  database: 'portal',
  define: {
    underscored: false,
    timestamps: false,
    freezeTableName: true
  },
  logging(...args) {
    console.log(args[0]);
  }
};
```

