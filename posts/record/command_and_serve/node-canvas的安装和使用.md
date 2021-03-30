---
title: node-canvas的安装和使用
date: 2017-09-13 15:26:51
tags: ["nodejs", "canvas"]
categories: ["记录"]
draft: true
---

## 关键词

* [node-canvas 官方windows安装教程](https://github.com/Automattic/node-canvas/wiki/Installation---Windows)
* [node-gyp 官方windows安装教程](https://github.com/nodejs/node-gyp#on-windows)

## windows环境下安装

`windows` 要使用 `node-canvas` 的话，要额外安装其他的工具或库来支持的，比如 `node-gyp` ，而安装 `node-gyp` 就要安装其他的工具，像 `Python2.7` ，一定要**2.7**版本，然后VC。

### 安装 `node-gyp`

这个现在就很简单了，我在网上看到的教程是以前的，安装的步骤可能比较复杂，而现在只需要一句命令，打开管理员的 `cmd`，输入 `npm install --global --production windows-build-tools` ，就会自动安装依赖了，包括 `Python` 和 `vc`

### 安装 `node-canvas` 

安装完 `node-gyp` 后，然后就要安装另外两个依赖， `GTK` 和 `libjpeg-turbo` ，这两个都要安装在**C盘**根目录

[GTK](http://ftp.gnome.org/pub/GNOME/binaries/win64/gtk+/2.22/gtk+-bundle_2.22.1-20101229_win64.zip)

然后添加环境变量：`C:\GTK\bin`

[libjpeg-turbo](https://sourceforge.net/projects/libjpeg-turbo/files/1.5.1/)，选择 `libjpeg-turbo-1.5.1-vc64.exe` 这个版本(64位)

最后，在管理员的 `cmd` 输入 `npm i canvas` 就完成了

## centos环境下安装（待测试）

`yum install cairo cairo-devel cairomm-devel libjpeg-turbo-devel pango-devel`

`cnpm i node-gyp -g`

`cnpm i node-gyp -d`

`cnpm i canvas -d`

完成！

### g++ not found

    yum -y update gcc
    yum -y install gcc+ gcc-c++

## node-canvas使用

