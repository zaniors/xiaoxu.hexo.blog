---
title: React如何开发开源UI组件（五）
date: 2020-04-22
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---

# 如何构建开发文档？React组件如何与文档集成？
可以参考Antd、ElementUI等，都会有个共同点，分开展示不同的属性下的按钮，以及所有Attribute展示接口

## 所存在的问题
- create-react-app创建的入口文件不适合组件管理展示
- 无法追踪调试以及行为属性模拟展示

## 使用StoryBook生成文档
我们熟悉的Antd是使用的[bisheng](https://github.com/benjycui/bisheng)这个工具生成的开发文档，bisheng是使用React将md文件转为静态文件站点的工具，网上也没有很全面的资料。

**[StoryBook](https://github.com/storybookjs/storybook)**，可以构建易管理，易测试的UI组件文档，本文将对[Button组件](/post/how-to-develop-a-react-component-2)为例来构建文档

### 安装StoryBook
```bash
MacBook-Pro:my-git zhangxu$ npx -p @storybook/cli sb init
MacBook-Pro:my-git zhangxu$ yarn storybook
```
#### npm script会多出来两条命令，本地运行、打包静态文件
```json
{
  "storybook": "start-storybook -p 9009 -s public",
  "build-storybook": "build-storybook -s public"
}
```

#### 配合Typescript使用
[storybook with ts 文档](https://storybook.js.org/docs/configurations/typescript-config/)
在新版的5.3.x中使用`npx -p @storybook/cli sb init`会检测到正在使用react或react-scripts，并安装适当的包，如果是create-react-app初始化项目下，会自动安装`@storybook/preset-create-react-app`，并且直接支持Typescript，不需要额外的配置

### 第一个Button文档
开始之前整理下目录结构，移除stories目录，默认创建的storybook会在根目录下创建stories，我这里将在各自组件下管理文档*.stories.tsx
storybook提供了三种写stories的方式：
- [x] StoriesOf API
- [x] Component Story Format (CSF)
- [x] MDX Syntax

#### StoriesOf API
这种写stories的方式是5.2之前的主要方式，我们来看下最简单的stories
```tsx
import { storiesOf } from '@storybook/react'
import React from 'react'
import Button from './button';

storiesOf('按钮组件', module)
  .add('默认按钮', () => <Button>默认按钮</Button>);
```

#### CSF
5.2+官方推荐的stories写法，里面必须有一个default export和一个或者多个named default，目前截止本篇文章时，CSF除了React Native外的所有框架都支持，RN还是建议使用StoriesOf
```tsx
import React from "react";
import Button from "./button";

export default {
  title: '按钮组件'
}
export const DefaultButton = () => <Button>默认按钮</Button>
```

整个storybook可以看作在写故事（其实和storiesOf差不多，只是CSF看上去更加清晰）：
**export default：**对故事描述书的元数据：title、decorators、parameters
**export named：**看作是故事书的某章节
> export named导出的故事章节将会使用Lodash的startCase函数包装，DefaultButton => Default Button

![storybook-button演示图](https://cdn.compelcode.com/image/fe/storybook-buitton-1.jpg)
这对应中文文档来说，是非常不友好的，但是又不可能将函数命名为中文导出，storybook可以对export named加入元数据name、decorators、parameters
```tsx
export const DefaultButton = () => <Button>默认按钮</Button>
DefaultButton.story = {
  name: '默认按钮'
}
```

#### 添加button样式
storybook提供几类修改配置的方式：
- main.js：常规配置，比如webpack等
- preview.js：配置decorators和parameters等
- manger.js：配置storybook ui的选项等

```tsx
// preview.js
import '../src/scss/index.scss';
```

![storybook-button演示图](https://cdn.compelcode.com/image/fe/storybook-buitton.jpg)

### StoryBook插件（addons）系统
**addons为storybook提供了额外的功能，有两种类型的插件**
1. Decorators：装饰器
2. Native Addons：自带的插件

#### Decorators是什么？如何使用？
decorator就像是自定义插件，它返回一个外层容器，最后在元数据里面使用。
```tsx
const centerTextStyle: React.CSSProperties = {
  textAlign: 'center'
}
const CenterTextDecorator = (storyFn: () => React.ReactNode) => <div style={centerTextStyle}>{storyFn()}</div>

export const DefaultButton = () => <Button onClick={action('clicked')}>默认按钮</Button>
DefaultButton.story = {
  name: '默认按钮',
  decorators: [CenterTextDecorator]
}

// storiesOf API
storiesOf('按钮组件', module)
  .add('默认按钮', () => <Button>默认按钮</Button>)
  .addDecorator(CenterTextDecorator);
```
> 其实就是作为外层容器把DefaultButton DOM包裹起来，如果有多个decorator，会一直嵌套下去

- addon-action：记录组件的行为

```tsx
import { action } from '@storybook/addon-actions';

export const DefaultButton = () => <Button onClick={action('clicked')}>默认按钮</Button>
DefaultButton.story = {
  name: '默认按钮'
}
```