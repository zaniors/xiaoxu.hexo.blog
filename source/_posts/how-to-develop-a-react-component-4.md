---
title: React如何开发开源UI组件（四）
date: 2020-04-17
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---

# 单元测试是什么？为什么需要单元测试？
![测试金字塔](https://cdn.compelcode.com/image/fe/front-end%20test%20pyramid.png)
- 单元测试：单元测试是整个金字塔中最基层，也是覆盖最大的地方，它是由最小逻辑单元组成，比如button、alert、menu组件等等，测试一个组件的测试用例，是否符合预期，比如Button组件：
  - 测试Button的默认class样式类名是否存在
  - 测试Button的自定义属性是否有效
  - 测试Button的点击事件是否有效
  - 测试Button的Link类型是否有效

# 测试我们Button逻辑单元
使用react-create-app脚手架创建的项目，jest可以开箱即用，测试框架有很多，除了API可能会不同，但整体解决方案大同小异。本文也将使用[react-testing-library](https://github.com/testing-library/react-testing-library)，react-testing-library只是一个模拟浏览器行为的解决方案，和[js-dom](https://jestjs.io/docs/en/tutorial-react)差不多

> 单页测试文件，按照之前的目录结构，放在和Button/下，\*.test.tsx or \*.spec.tsx都可以


