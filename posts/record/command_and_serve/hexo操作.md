---
title: hexo操作
date: 2017-08-02 11:42:13
tags: ["hexo"]
categories: ["记录"]
draft: true
---

> 记录平常常用的hexo操作

# hexo安装

    // 在全局安装hexo
    npm i hexo -g

# 更换主题

1. 先在博客根目录里，`git clone`你要安装的主题
2. 在博客的`_config.yml`配置文件中，修改`themes`
3. 完成

每个主题，作者都会有配置介绍的，根据配置来调整就好了。

# 命令

1. hexo n 'name' -- 新建文章
2. hexo clean -- 清除缓存（根据网上文章，每次上传前，都要先清除缓存）
3. hexo s -- 开发模式
4. hexo g -- 生成文件
5. hexo d -- 上传文件

# 设置分类

每篇文章中的tag下面加上`categories:`，就会自动生成你想要的分类了，还可以在根目录文件中，`scaffolds/post.md`加上`categories:`，这样就会自动添加分类了。
