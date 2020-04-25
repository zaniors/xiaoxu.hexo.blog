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
而React则使用Gatsby构建的静态内容

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
```ts
import { storiesOf } from '@storybook/react'
import React from 'react'
import Button from './button';

storiesOf('按钮组件', module)
  .add('默认按钮', () => <Button>默认按钮</Button>);
```

#### CSF
5.2+官方推荐的stories写法，里面必须有一个default export和一个或者多个named default，目前截止本篇文章时，CSF除了React Native外的所有框架都支持，RN还是建议使用StoriesOf
```ts
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
这对于中文文档来说，是非常不友好的，但是又不可能将函数命名为中文导出，storybook也可以对export named加入元数据name、decorators、parameters
```ts
export const DefaultButton = () => <Button>默认按钮</Button>
DefaultButton.story = {
  name: '默认按钮'
}
```

#### MDX语法
这种写法需要单独安装包和配置，可以在文档内写md语法，[MDX README](https://github.com/storybookjs/storybook/tree/next/addons/docs/react)
截止此文章发布的时候MDX的官网文档还在更新中...

#### 添加button样式
storybook提供几类修改配置的方式：
- main.js：常规配置，比如webpack等
- preview.[jt]s：配置decorators和parameters等
- manger.[jt]s：配置storybook ui的选项等
> 由于项目使用的ts，添加decorators和parameters需要代码提示，可以将后缀改为ts

```ts
// preview.ts
import '../src/scss/index.scss';
```

![storybook-button演示图](https://cdn.compelcode.com/image/fe/storybook-buitton.jpg)

### StoryBook插件（addons）系统
**addons为storybook提供了额外的功能，有两种类型的插件**
1. Decorators：装饰器
2. Native Addons：自带的插件

#### Decorators
Decorator就像是自定义插件，它返回一个外层容器，最后在元数据里面使用
```ts
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

当然也可以全局配置Decorators
```ts
// preview.ts
import React from 'react';
import { addDecorator } from '@storybook/react';
import '../src/scss/index.scss';

const centerTextStyle = {
  textAlign: 'center'
}
const CenterTextDecorator = (storyFn) => <div style={centerTextStyle}>{storyFn()}</div>

addDecorator(CenterTextDecorator)
```

#### Native Addons
- addon-action：记录组件的行为
- addon-link：在storybook中导航

```ts
// .storybook/main.js 注册Addons
module.exports = {
  addons: [
    '@storybook/addon-actions',
    '@storybook/addon-links',
  ],
};
```

**看看action如何使用**
```ts
import { action } from '@storybook/addon-actions';

export const DefaultButton = () => <Button onClick={action('clicked')}>默认按钮</Button>
```

```ts
import { actions } from '@storybook/addon-actions';

const eventsFromNames = actions({ onClick: 'clicked', onMouseOver: 'hovered' });

export const DefaultButton = () => <Button {...eventsFromNames}>默认按钮</Button>
```

**看看link如何使用**
linkTo(idOrKindInput,  storyInput)：接受两个参数
idOrKindInput：export default的元数据`title`
storyInput：export nemed的元数据`name`
```ts
import { linkTo } from '@storybook/addon-links';

export const DefaultButton = () => <Button onClick={linkTo('按钮组件', '按钮尺寸')}>跳转文档</Button>
```

storybook中的Addons可以实现丰富的展示效果
- Knobs：允许动态修改组件的值
- Source：允许在Panel中显示源代码
- Info：允许显示一些额外的信息
- ...

### 利用addon-info完善更丰富的文档
安装addon-info
```bash
MacBook-Pro:my-git zhangxu$ yarn add @storybook/addon-info -D
MacBook-Pro:my-git zhangxu$ yarn add @types/storybook__addon-info -D
```
> 提示：在addons中提供三种级别去使用，全局 < 故事级（export default）< 章节级（export named），named也是权重最高的

这里使用全局配置通用的addon-info
```ts
import { addDecorator, addParameters } from '@storybook/react';
import { withInfo } from '@storybook/addon-info';
import '../src/scss/index.scss';

addDecorator(withInfo);
addParameters({
  info: {
    inline: true //如何展示额外的信息，true直接展示在页面上，false通过button触发展示
  }
})
```

button组件里额外配置相关信息
333
```ts
// button.stories.tsx export default
export default {
  title: '按钮组件',
  component: Button,
  parameters: {
    info: {
      text: '默认按钮用于没有主次之分的一组行动点'
    }
  }
}
```
```ts
// button.stories.tsx export named config
export const DefaultButton = () => <Button onClick={action('clicked')}>默认按钮</Button>
DefaultButton.story = {
  name: '默认按钮',
  parameters: {
    info: {
      text: '默认按钮用于没有主次之分的一组行动点'
    }
  }
}
```
> text支持markdown语法

具体的配置文档，👉[参考](https://github.com/storybookjs/storybook/tree/master/addons/info)

目前整体效果还不错，但是表格中的属性和描述等手写非常费时间，可否自动判断代码注释来生成文档
![storybook-button演示图](https://cdn.compelcode.com/image/fe/storybook-buitton-2.jpg)

### 通过代码注释来自动生成文档
```bash
MacBook-Pro:my-git zhangxu$ yarn add react-docgen-typescript-loader -D
```
```ts
// main.js
```