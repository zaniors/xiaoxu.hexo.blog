---
title: 制作icon的几种方式
date: 2019-04-07 09:32:53
thumbnail: http://cdn.compelcode.com/image/fe/font-icon.jpg
tags: [前端,CSS]
categories: [前端]
---
在前端里面通常会制作自己的icon图标，以前可能是用图片来制作，缺点肯定已经很明显，体积大，不易维护，每个图片一次请求等等

>#### base64

#### 将一副图片的数据编码成一串字符串，使用该字符串代替图片地址
- CSS中使用: background: url(data: image/{ type };base64, { img-data });
- HTML中使用: img标签：src="data: image/{ type };base64, { img-data }"

#### 语法: ``` data: image/{ type };base64, { img-data } ```
- type：表示图片的格式，png、jpg、svg...
- img-data：表示图片转换的图片数据，可以百度转换工具

#### 优点
- 一个base64可以节省一HTTP请求
- 加载速度快

#### 缺点
- 体积被编码后将会变大，但是通过gzip压缩后，也会优化很多
- 编码之后字符串很多，将代码变得冗余，而且编辑器的加载也会影响

#### 与CSS Sprite目的差不多，都是为了减少请求，两者应如何取舍？

- 如果图片足够小，没办法制作成雪碧图，整个网站的复用性很高，且基本不会被更新可以考虑base64，比如背景图，制作2*2的图片，上下平铺作为背景图的情况，就无法做成雪碧图，但又为了减少请求，base64的优化就非常适合


> #### CSS3

  - 只适合制作一些有规则的图形上
  - 优点：对于简单的图形，可能用更少的代码就能完成，占用更少的空间
  - 缺点：不利于维护，可能对于PC兼容性不太好，手机端兼容性几乎不考虑

> #### IconFont
  这里使用[IcoMoon](https://icomoon.io/)来制作
  
  - 在网上下载好你需要的SVG图片，比如([阿里矢量图库](http://www.iconfont.cn)、[easyicon](http://www.easyicon.net/))，也可以在IconMoon选择你要需要的icon

  - 在IconMoon官网右上角-IcoMoon APP-important Icons，导入你现在的SVG图片

  - 值得注意的是，生成字体文件之前，最好设置下font-name、class-prefix等等
> 提示：可以直接使用[阿里矢量图库](http://www.iconfont.cn)来直接制作，其都差不多
