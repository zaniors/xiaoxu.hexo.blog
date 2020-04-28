---
title: React如何开发开源UI组件（六）
date: 2020-04-27
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---
# 前言
之前已经完成一个Button组件，包括样式，测试以及开发文档等等，但是作为开源组件还需要和全世界程序员开心快乐的使用，发布npm

前端的模块化：
- 全局模块规范：例如，jQuery通过暴露全局的$来访问内部模块方法。全局引入js的弊端：全局变量污染、内部依赖顺序难以维护、无法自动化处理依赖模块等
- CommonJS：Node推出在服务端的模块化规范
- AMD规范：将模块化规范推行到浏览器端，解决了早期全局引入js的所有问题，可以配合打包工具（webpack、rollup等），但是AMD语法规范在浏览器端使用还得用RequireJs来运行
- UMD规范：这种规范更像一个语法糖，它希望支持以上三种规范，大概就是在执行UMD规范时，会优先判断是当前环境是否支持AMD环境，然后再检验是否支持CommonJS环境，否则认为当前环境为浏览器环境（ window ）
- ES Modules：ES模块是官方标准，也是JavaScript语言明确的发展方向，而CommonJS模块是一种特殊的传统格式，在ES模块被提出之前做为暂时的解决方案。 ES模块允许进行静态分析，从而实现像 tree-shaking 的优化，并提供诸如循环引用和动态绑定等高级功能

> 引出一个问题，应该提供给使用者以何种方式引入我们的组件
参考antd文档看，提供两种方式引入
1. 使用npm或者yarn来安装（ES Module）
2. 浏览器引入，也就是已构建的UMD格式

# 利用TSC编译器打包JS模块

我们用create-react-app创建的入口文件不适合组件管理，展示webapp的入口页面
## 整理组件的index.tsx入口文件
```ts
export { default as Button } from './components/Button';
```

## 添加Button/index.tsx组件默认导出
```ts
import Button from './button';

export default Button;
```

## 配置tsconfig.build.json
```json
{
  "compilerOptions": {
    "outDir": "dist",
    "module": "esnext",
    "target": "es5",
    "declaration": true,
    "jsx": "react",
    "moduleResolution": "Node",
    "allowSyntheticDefaultImports": true
  },
  "include": [ 
    "src"
  ],
  "exclude": [
    "src/**/*.stories.tsx",
    "src/**/*.test.tsx",
    "src/setupTests.ts"
  ]
}
```

## 配置npm script
```json
{
    "build": "npm run build-ts && npm run build-css",
    "build-ts": "tsc -p tsconfig.build.json",
    "build-css": "node-sass ./src/scss/index.scss ./dist/index.css",
}
```

## 借助rimraf第三方库，在build前移除dist目录
```bash
yarn add rimraf -D
```
```json
{
  "clean": "rimraf ./dist",
  "build": "npm run clean && npm run build-ts && npm run build-css"
}
```

## 配置package.json
```json
{
  "main": "dist/index.js", // 模块的主入口文件
  "module": "dist/index.js", // ES Module主入口文件
  "type": "dist/index.d.ts", // TS Typing文件
}
```

这里**module**字段并不属于package.json的规则，而是rollup提出[pkg.module](https://github.com/rollup/rollup/wiki/pkg.module)的概念，最开始npm都是基于CommonJS规范来着的，main.js作为模块require的主入口，而ES Module没出来之前，一直用CommonJS作为临时方案，而现在rollup构建ES Module时，可以受益tree-shaking的特性提高性能，官方不希望和CommonJS使用同一字段**main**作为入口，rollup则使用了**module**作为ES Module识别入口，webpack2开始也支持pkg.module

## 大功告成
```bash
npm publish 
```
发布之前npm script提供一个钩子，帮助我们build最新代码
```json
{
  "prePublish": "npm run build"
}
```
