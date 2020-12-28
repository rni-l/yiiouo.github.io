---
title: babel-loader解析 vue
date: 2020-12-02 12:00:00
tags: [“webpack", “babel"]
categories: ["记录"]

---

> 在用 React 的脚手架开始一个项目，有个需求需要用到 Vue，生成 Vue 文件。因为不想再重新开一个项目，所以直接在这个 React 项目继续开发



## vue-loader

要解析 `vue` 的单文件模板，就一定要用到 [vue-loader](https://vue-loader.vuejs.org/zh/guide/)。很简单，跟着教程来就行了，这里不再做说明



## babel-loader

因为我是用 `ts` 去写 `vue` 文件的，所以还要解析 `ts`。`vue-loader` 文档是用 `ts-loader` 进行解析，但我这个项目用的是 `babel-loader` 解析 `ts`，而不是 `ts-loader`。如果按照文档去配置，运行时会报错。

先贴下项目是怎么用 `babel-loader` 解析 `ts`：

```javascript
{
  test: /\.tsx?$/,
  exclude: /node_modules/,
  loader: 'babel-loader',
  options: {
    cacheDirectory: true,
  },
}

{
  presets: [
    // @babel/preset-env will automatically target our browserslist targets
    require('@babel/preset-env'),
    [
      require('@babel/preset-typescript'),
      {
        isTSX: true,
        allExtensions: true, // 解析所有后缀
      },
    ],
    [require('@babel/preset-react'), { development }],
  ],
  plugins: [
    // Stage 0
    require('@babel/plugin-proposal-function-bind'),

    // Stage 1
    require('@babel/plugin-proposal-export-default-from'),
    require('@babel/plugin-proposal-logical-assignment-operators'),
    [require('@babel/plugin-proposal-optional-chaining'), { loose: false }],
    [
      require('@babel/plugin-proposal-pipeline-operator'),
      { proposal: 'minimal' },
    ],
    [
      require('@babel/plugin-proposal-nullish-coalescing-operator'),
      { loose: false },
    ],
    require('@babel/plugin-proposal-do-expressions'),

    // Stage 2
    [require('@babel/plugin-proposal-decorators'), { legacy: true }], // 解析“装饰器”
    require('@babel/plugin-proposal-function-sent'),
    require('@babel/plugin-proposal-export-namespace-from'),
    require('@babel/plugin-proposal-numeric-separator'),
    require('@babel/plugin-proposal-throw-expressions'),

    // Stage 3
    require('@babel/plugin-syntax-dynamic-import'),
    require('@babel/plugin-syntax-import-meta'),
    [require('@babel/plugin-proposal-class-properties'), { loose: true }], // 配置解析类的“装饰器”
    require('@babel/plugin-proposal-json-strings'),
  ],
}

```

在它的原有基础上，我加了：

```javascript
{
  isTSX: true,
  allExtensions: true, // 解析所有后缀
},
```

就可以了。现在这套配置，支持 `React` 和 `Vue`



## 链接：

1. [vue-loader](https://vue-loader.vuejs.org/zh/guide/pre-processors.html#typescript)
2. [babel](https://babeljs.io/docs/en/babel-preset-typescript#allowdeclarefields)
3. [教程](https://segmentfault.com/q/1010000019894930)
4. [解决 @ 报错问题](https://github.com/kaorun343/vue-property-decorator/issues/26)