---

title: babel
date: 2018-12-11
tags: [“js”, “babel"]
categories: ["记录"]
draft: true
---

## Babel

### 资料链接

1. [中文网](https://www.babeljs.cn/docs/next/)
2. [babel-transform](https://www.babeljs.cn/docs/next/babel-core#transform)
3. [babel-parser](https://www.babeljs.cn/docs/next/babel-parser)
4. [babel-generator](https://www.babeljs.cn/docs/next/babel-generator)
5. [babel-options](https://www.babeljs.cn/docs/next/options#code)
6. [babel-types](https://babeljs.io/docs/en/next/babel-types.html)

### transform 和 parser 区别

transform 会根据你配置的 plugins ，将 code 处理并输出

parser 是将 code 返回 ast 数据

简单来讲：transform 是 parser 的强化版

### ast

babel 转换原理：

字符串 code  -> 通过 trasnfrom/parse 解析成 ast -> 根据你配置的插件，将 ast 里面的值进行替换 -> 通过 generator 将 ast 转为 code