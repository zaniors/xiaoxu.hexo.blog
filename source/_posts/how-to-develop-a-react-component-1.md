---
title: React如何开发开源UI组件（一）
date: 2020-04-10
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---

# 前言
## 本文将一步一步的记录如何开发UI组件，开源的UI组件具体需要注意什么，有哪些步骤？并且将以Button简单的组件为例子

### 思考问题
- 项目代码的组织结构
- 组件需求分析
- 采用怎样的样式解决方案
- 如何写组件单元的逻辑测试
- 如何构建开发文档
- 如何打包发布，以及自动化测试

### 开发步骤
- [x] 使用React Hook加上Typescript助力，虽然两者都不是必选，但是可以让我们的代码更加清晰，毕竟代码是给人看的
  - Hook是React16.8新特性，可以让逻辑更好的复用，更好的理解
  - Typescript的好处不用多说，类型检测、代码接口提示、泛型、枚举等等，比如难以跟踪的运行时异常、组件接口具体提示和描述等等
- [x] 使用Scss/Less来处理样式，定义变量、函数等，以及构建复杂的逻辑
- [x] 使用单元测试，如果使用create-react-app创建的项目，那么无缝使用Jest来测试我们的代码逻辑，可以用[js-dom](https://jestjs.io/docs/en/tutorial-react)来操作DOM;或者使用[React测试库](https://testing-library.com/docs/react-testing-library/intro)也是本文将采用的测试方式
- [x] 构建开发文档，不论是自己维护迭代还是提供他人观看，文档肯定必不可少
- [x] 利用tsc（Typescript CLI）进行模块化打包，生成ES6 Module
- [x] 结合Travis CI/CD，持续集成和部署，最后发布NPM使用