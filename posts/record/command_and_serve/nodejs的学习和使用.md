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



### 设置源

```shell
npm set registry https://registry.npm.taobao.org # 注册模块镜像
npm set disturl https://npm.taobao.org/dist # node-gyp 编译依赖的 node 源码镜像

# 设置 npm 源
npm set registry https://registry.npmjs.org
npm set registry http://47.107.21.127:4873
npm set registry http://10.10.5.63:4873

## 以下选择添加
npm set sass_binary_site https://npm.taobao.org/mirrors/node-sass # node-sass 二进制包镜像
npm set electron_mirror https://npm.taobao.org/mirrors/electron/ # electron 二进制包镜像
npm set puppeteer_download_host https://npm.taobao.org/mirrors # puppeteer 二进制包镜像
npm set chromedriver_cdnurl https://npm.taobao.org/mirrors/chromedriver # chromedriver 二进制包镜像
npm set operadriver_cdnurl https://npm.taobao.org/mirrors/operadriver # operadriver 二进制包镜像
npm set phantomjs_cdnurl https://npm.taobao.org/mirrors/phantomjs # phantomjs 二进制包镜像
npm set selenium_cdnurl https://npm.taobao.org/mirrors/selenium # selenium 二进制包镜像
npm set node_inspector_cdnurl https://npm.taobao.org/mirrors/node-inspector # node-inspector 二进制包镜像

npm cache clean --force # 清空缓存
```





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



## 搭建私有 npm

安装包：npm install -g verdaccio --unsafe-perm

启动：verdaccio

