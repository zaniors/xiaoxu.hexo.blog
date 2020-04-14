---
title: React如何开发开源UI组件（二）
date: 2020-04-12
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---

# 创建属于自己的Button组件
```bash 
MacBook-Pro:my-git zhangxu$ npx create-reatc-app UIReact --typescript
```
## 分析我们的目录结构，是否有一种完美的标准目录结构？
[可以参考React官方文档](https://react.docschina.org/docs/faq-structure.html)，答案是否定的，所以按照自己项目搭配，设计目录时只要遵循以下：避免多层嵌套，不要过度思考
![目录结构](https://cdn.compelcode.com/image/fe/bc-file-structure.png)

## 安装normalize.css来重置浏览器样式保持一致性
> [https://github.com/necolas/normalize.css/blob/master/normalize.css](https://github.com/necolas/normalize.css/blob/master/normalize.css)

## 分析Button功能
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

## 疑难问题
- 两种按钮类型，存在button和a元素，
  - disabled属性存在时，对button设置原生属性，对a则设置disabled的class类名
  - href属性存在时，那么则返回a元素，否则即使type为link，那么也是button元素
- 原生属性如何使用？比如onClick事件，button的name属性，a的target属性等

## 定义接口
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
> 这里使用了<span style="color: red; font-weight: bold;">字符串字面量</span>配合<span style="color: red; font-weight: bold;">联合类型</span>，定义了ButtonType，这样的好处是什么呢？为什么不使用enum或者interface

使用enum和interface都是没有问题的，但是有一点就是，接口需要export出去，调用者在使用时需要import接口，比如：
```tsx
// button.tsx
export enum ButtonType {
  Primary = 'primary',
  Default = 'default',
  Danger = 'danger',
  Link = 'link'
}

// App.tsx
import Button, { ButtonType } from './components/Button/button';

<Button type={ButtonType.Link} disabled>按钮</Button>
```

## 在React中添加动态的class，变得很繁琐，推荐使用classnames第三方库编写动态class
> [classnames => usage-with-reactjs](https://www.npmjs.com/package/classnames#usage-with-reactjs)
解决问题一：
```tsx
// size属性为'large' => g-btn-large
// type属性为'primary' => g-btn-primary
// href属性为'https://xxx.com' => disabled
// 最终classes为：'g-btn g-btn-large g-btn-primary disabled'

const classes = classNames(prefixCls, {
    [`g-btn-${size}`]: size,
    [`g-btn-${type}`]: type,
    disabled: href && type === 'link' && disabled,
  }, className)

return (
  href && type === 'link' ?
    <a className={classes} href={href} {...rest}>{children}</a>
    :
    <button disabled={disabled} className={classes} {...rest}>{children}</button>
)
```

## 借助TS的<span style="color: red; font-weight: bold;">交叉类型<span><span style="color: black; font-weight: 700;">，将DOM元素的原生属性和我们自定义属性进行合并</span>
```tsx
import React, { FC, ButtonHTMLAttributes, AnchorHTMLAttributes } from 'react';

type INativeButtonProps = ButtonHTMLAttributes<HTMLElement>;
type INativeAnchorProps = AnchorHTMLAttributes<HTMLElement>;

type ButtonProps = INativeButtonProps & INativeAnchorProps & IBaseButtonProps;

const Button: FC<ButtonProps> = (props) => { ... }
```
写到这里想必是大功告成！！！终于...
等等！为什么无法使用type属性了，为什么代码报错了！
![type-error](https://cdn.compelcode.com/image/fe/bc-type-error.png)

我们去看看React.ButtonHTMLAttributes的定义
```tsx
// node_modules/@types/react/index.d.ts
interface ButtonHTMLAttributes<T> extends HTMLAttributes<T> {
  autoFocus?: boolean;
  disabled?: boolean;
  form?: string;
  formAction?: string;
  formEncType?: string;
  formMethod?: string;
  formNoValidate?: boolean;
  formTarget?: string;
  name?: string;
  type?: 'submit' | 'reset' | 'button';
  value?: string | string[] | number;
}
```
如你所看到的，原生button已经有type定义，那非要使用type属性怎么做？可否忽略掉原生的type属性？答案是肯定的。借助TS的<span style="color: red; font-weight: bold;">Omit类型</span>
我们重新定义一下ButtonProps接口
```tsx
type ButtonProps = Omit<INativeButtonProps, 'type'> & INativeAnchorProps & IBaseButtonProps;
```
Omit高级类型的源码：
```tsx
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```
Pick：挑选出T类型中K的属性 => Pick<{ name: string; age: number }, 'name> => // { name: string }
Exclude：排除T中含有K属性的，返回T => Exclude<string | number, number> => // string
Omit：忽略某些属性，Omit<{ name: string, age: number }, 'name'> => // { age: number }

[查看source code](https://github.com/ZAnsder/UIReact/blob/master/src/components/Button/button.tsx)