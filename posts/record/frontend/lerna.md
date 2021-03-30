---
title: lerna
date: 2020-04-10 09:06:00
tags: [“lerna”, “npm"]
categories: ["记录"]
draft: true
---

## 参考链接

1. [https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/665469/](https://codertw.com/程式語言/665469/)
2. [https://medium.com/lion-f2e/lerna-js-package-%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7-e9ed360d1143](https://medium.com/lion-f2e/lerna-js-package-管理工具-e9ed360d1143)
3. [juejin](https://juejin.im/post/5a989fb451882555731b88c2#heading-1)
4. [中文文档]([https://github.com/chinanf-boy/lerna-zh#%E5%85%A5%E9%97%A8](https://github.com/chinanf-boy/lerna-zh#入门))



## 快速开始

全局安装

```shell
npm i lerna -g
```

建文件夹，初始化

```shell
mkdir test && cd test
lerna init
git init
git remote add origin ...

```

添加模块

```shell
lerna create test1 -y
lerna create test2 -y
```

发布模块，不然包之间无法引用

```shell
cd ./packages/test1
// 修改好 package.json 的信息...
npm publish
```

将 test1 模块添加到 test2

```shell
// 使用 lerna add 的方式添加，会自动生成软链接，test1 模块修改后，test2 自动会更新
lerna add test1 --scope=test2

// 手动添加依赖
cd ./packages/test1
npm i test1 -S
// 生成软链接
lerna bootstrap
```

发布模块

```shell
// 会把当前更新了的模块，都进行发布
lerna publish
```



