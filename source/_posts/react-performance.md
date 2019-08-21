---
title: React中记录性能优化
date: 2019-05-06 12:26:01
tags: [react]
categories: [前端]
thumbnail: https://cdn.compelcode.com/image/fe/react-performance.jpg
---

总结在react中的性能优化
## 将事件绑定实例写到constructor中

> 程序一开始就绑定this，提升性能。如果使用箭头函数则会每次程序会去找this作用域或者在render中bind(this)，每次state变化时重新执行render都会绑定作用域也会耗低性能

``` jsx
constructor() {
    this.handleClick = this.handleClick.bind(this);
}
```

## diff算法对比，减少update次数
> key属性，值要唯一

> 官方所提出的setState异步更新，正常运行时就是批更新，达到一定条件将队列中多次更改合并一次，触发一次更新state

> 父组件的render，会触发子组件update(无论props是否更新)，可以在子组件shouldComponentUpdate判断是否更新子组件

```jsx
shouldComponentUpdate(nextProps, netState) {
    if (nextProps.xxx !== this.props.xxx) {
        return true;
    }
    return false;
}
```