---
title: React如何开发开源UI组件（二）
date: 2020-04-12
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---

### 创建属于自己的Button组件
```bash 
MacBook-Pro:my-git zhangxu$ npx create-reatc-app UIReact --typescript
```
#### 分析我们的目录结构，是否有一种完美的标准目录结构？
[可以参考React官方文档](https://react.docschina.org/docs/faq-structure.html)，答案是否定的，所以按照自己项目搭配，设计目录时只要遵循以下：避免多层嵌套，不要过度思考
![目录结构](https://cdn.compelcode.com/image/fe/bc-file-structure.png)

#### 安装normalize.css来重置浏览器样式保持一致性
> [https://github.com/necolas/normalize.css/blob/master/normalize.css](https://github.com/necolas/normalize.css/blob/master/normalize.css)

#### 分析Button功能
- [x] Button Type： 按钮的类型
  - Primary（主按钮）：主要按钮，一个区域内拥有一个主按钮
  - Default（默认）：默认按钮，主次不明确情况下展示默认按钮
  - Danger（危险）：危险按钮，用于警告或者权限等危险操作，一般需要二次确认
  - Link（链接）：链接按钮，保持和原生A标签一样的操作，需配合href可跳转
- [x] Button Size：按钮的尺寸
  - large：大号尺寸
  - middle：中号尺寸（默认）
  - small：小号尺寸
- [x] Button Disabled：按钮禁用属性
- [x] Button Href：按钮跳转链接（Link有效）

#### 疑难问题
- 两种按钮类型，存在button和a元素，
  - disabled属性存在时，对button设置原生属性，对a则设置disabled的class类名
  - href属性存在时，那么则返回a元素，否则即使type为link，那么也是button元素

#### 定义接口
```tsx
export type ButtonType = 'primary' | 'default' | 'danger' | 'link';
export type ButtonSize = 'large' | 'middle' | 'small';

interface IBaseButtonProps {
  size?: ButtonSize;
  type?: ButtonType;
  disabled?: boolean;
  href?: string;
  className?: string;
  children?: React.ReactNode;
}
```

#### 在React中添加动态的class，变得很繁琐，推荐使用classnames第三方库编写动态class
> [classnames => usage-with-reactjs](https://www.npmjs.com/package/classnames#usage-with-reactjs)

```tsx
// size属性为'large' => g-btn-large
// type属性为'primary' => g-btn-primary
// href属性为'https://xxx.com' => disabled
// 最终classes为：'g-btn g-btn-large g-btn-primary disabled'

const prefixCls = 'g-btn';
const classes = classNames(prefixCls, {
    [`${prefixCls}-${size}`]: size,
    [`${prefixCls}-${type}`]: type,
    disabled: href && disabled,
  }, className)
```

#### 