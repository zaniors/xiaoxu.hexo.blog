---
title: React如何开发开源UI组件（三）
date: 2020-04-15
tags: [前端,React,UI,Component]
categories: [前端]
thumbnail:
---

# SCSS预处理器为我们的组件样式助力
安装node-sass，React脚手架默认没有支持css预处理器
```bash
MacBook-Pro:UIReact zhangxu$ yarn add node-sass
```

```
UIReact/
  src/
    components/
      Button/
        styles/
          _base.scss
          _mixin.scss
          _index.scss
        ...
    scss/
      _variables.scss
      _reboot.scss
      _mixin.scss
      index.scss
    ...
```
- scss/_variables.scss 放置全局通用的变量
- scss/_reboot.scss 对normalize.css二次封装，加入一些定义的变量
- scss/_mixin.scss 放置全局的mixin函数
- Button/styles/_base.scss Button组件样式变量
- Button/styles/_mixin.scss Button组件的mixin函数

> 导入sass/scss文件使用Partials（不编译为css文件，导入合并），也就是sass/scss文件加上<span style="color: red; font-weight: bold;">_index.scss</span>下划线

## 如何结合Scss动态生成不同的样式
我们知道在Button.tsx中，传入不同的属性会生成不同的className，比如：
```html
<Button>默认按钮</Button> // g-btn g-btn-md g-btn-default
```
- g-btn：Button的一些基本样式
- g-btn-md：Button尺寸，默认中号
- g-btn-default：Button的默认样式

结合Scss，定义不同按钮样式变量和边距大小，传入到mixin函数中

## 定义Button的样式变量
```scss
// 按钮基本属性
$btn-font-weight:             400;
$btn-font-family:             $font-family-base !default;
$btn-line-height:             $line-height-base !default;
$btn-border-width:            $border-width !default;
$btn-box-shadow:              inset 0 1px 0 rgba($white, .15), 0 1px 1px rgba($black, .075) !default;
$btn-transition:              color .15s ease-in-out, background-color .15s ease-in-out, border-color .15s ease-in-out, box-shadow .15s ease-in-out !default;

// 不同大小按钮的 padding 和 font size 和 border-radius
// 默认按钮的样式
$btn-padding-y:               .375rem !default;
$btn-padding-x:               .75rem !default;
$btn-font-size:               $font-size-base !default;
$btn-border-radius:           $border-radius !default;
```

## 定义Button的Mixin函数
```scss
@mixin button-size($btn-padding-y, $btn-padding-x, $btn-font-size, $border-radius) {
  padding: $btn-padding-y $btn-padding-x;
  font-size: $btn-font-size;
  border-radius: $border-radius;
}

@mixin button-style($color,
  $border,
  $background,
  $hover-color: $color,
  $hover-border: lighten($border, 10%),
  $hover-background: lighten($background, 7.5%)) {
  color: $color;
  border-color: $border;
  background: $background;

  &:hover {
    color: $hover-color;
    border-color: $hover-border;
    background: $hover-background;
  }

  &:focus,
  &.focus {
    color: $hover-color;
    border-color: $hover-border;
    background: $hover-background;
    outline: none;
  }

  &:disabled,
  &.disabled {
    color: $color;
    border-color: $border;
    background: $background;
    outline: none;
  }
}
```

## 如何使用？
```scss
@import 'mixin', 'base';

.g-btn {
  position: relative;
  display: inline-block;
  font-weight: $btn-font-weight;
  line-height: $btn-line-height;
  color: $body-color;
  white-space: nowrap;
  text-align: center;
  vertical-align: middle;
  background-image: none;
  border: $btn-border-width solid transparent;
  box-shadow: $btn-box-shadow;
  transition: $btn-transition;
  cursor: pointer;

  &-md {
    @include button-size($btn-padding-y, $btn-padding-x, $btn-font-size, $border-radius);
  }

  &-default {
    @include button-style($body-color, $gray-400, $white, $primary, $primary, $white);
  }
}
```

[查看完整的source code](https://github.com/ZAnsder/UIReact/blob/master/src/components/Button/styles/_index.scss)