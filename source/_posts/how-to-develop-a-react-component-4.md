---
title: React如何开发开源UI组件（四）
date: 2020-04-17
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---

# 单元测试是什么？为什么需要单元测试？
![测试金字塔](https://cdn.compelcode.com/image/fe/front-end%20test%20pyramid.png)
- 单元测试：单元测试是整个金字塔中最基层，也是覆盖最大的地方，它是由最小逻辑单元组成，比如button、alert、menu组件等等，测试一系列逻辑是否符合预期，比如Button组件：
  - 测试Button的默认class样式类名是否存在
  - 测试Button的自定义属性是否有效
  - 测试Button的点击事件是否有效
  - 测试Button的Link类型是否有效

# 测试我们Button逻辑单元
使用react-create-app脚手架创建的项目，jest可以开箱即用，测试框架有很多，除了API可能会不同，但整体解决方案大同小异。本文也将使用[react-testing-library](https://github.com/testing-library/react-testing-library)，react-testing-library只是一个模拟浏览器行为的解决方案，和[js-dom](https://jestjs.io/docs/en/tutorial-react)差不多

> 新建单元测试文件，按照之前的目录结构，放在和Button/下，\*.test.tsx or \*.spec.tsx都可以

```ts
describe('1 + 1的结果', () => {
  test('测试1 + 1是否等于2', () => {
    expect(1 + 1).toBe(2);
  })
})
```
- jest.describe：一组测试用例的集合
- jest.test：jest.it的别名，某个逻辑的测试用例
- jest.expect：断言我们的逻辑是否符合预期

```ts
import React from 'react';
import { render } from '@testing-library/react';
import Button from './button';

describe('测试Button组件', () => {
  it('测试Button的默认class样式类名是否存在', () => {
    const { getByText } = render(<Button>默认按钮</Button>)
    const defaultBtn = getByText('默认按钮')

    expect(defaultBtn).toBeInTheDocument()  // 是否在document中存在
    expect(defaultBtn).toHaveClass('g-btn g-btn-default g-btn-md') // 默认的class是否存在
  })
})
```

**当我们每写完一个测试用例，我们的代码就会更加的健壮，无论是迭代开发新功能，还是团队成员维护，这些测试用例都会给我们带来诸多好处，比如不小心删除了某些逻辑，初始化g-btn没有添加，这段测试用例则不会通过，我们可以很快的定位到问题所在，并且后面配合npm script和Travis CI，可以在提交代码时运行测试，通过才能打包发布等等**

[查看完整的source code](https://github.com/ZAnsder/UIReact/blob/master/src/components/Button/button.test.tsx)
[jest完整的断言文档](https://jestjs.io/docs/en/expect#reference)
