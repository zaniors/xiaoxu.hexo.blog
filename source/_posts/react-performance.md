---
title: react-performance
date: 2019-05-06 12:26:01
tags: [react]
categories: [前端]
thumbnail: http://cdn.compelcode.com/image/fe/react-performance.jpg
---

总结在react中的性能优化
## 将事件绑定实例写到constructor中

> 程序一开始就绑定this，提升性能。如果使用箭头函数则会每次程序会去找this作用域或者在render中bind(this)，每次state变化时重新执行render都会绑定作用域也会耗低性能

``` jsx
constructor() {
    this.handleClick = this.handleClick.bind(this);
}
```
