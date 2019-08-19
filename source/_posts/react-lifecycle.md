---
title: React生命周期
date: 2019-04-29 09:48:40
tags: [react]
categories: [前端]
thumbnail: http://cdn.compelcode.com/image/fe/react-lifecycle.png
---

不管学习任何框架，掌握框架的生命周期函数，才知道组件在某一时刻在执行什么任务

## React的生命周期主要分为三大块
![react-lifecycle-v16.3](http://cdn.compelcode.com/image/fe/react-lifecycle-v16.3.png)(React16.3)
![react-lifecycle-v16.4](http://cdn.compelcode.com/image/fe/react-lifecycle-v16.4.png)(React16.4)

### Mounting
- **constructor**
> 组件挂载之前，调用构造函数。给this.state赋值来初始化、给事件函数绑定this
- static getDerivedStateFromProps
- **render**
> 渲染组件
- **componentDidMount**
> 组件被挂载DOM树后，和DOM相关的操作应在这里
已废弃
- ~~componentWillMount~~ 

### Update
- static getDerivedStateFromProps
> 在组件render前，首次创建和更新时都会被调用
- **shouldComponentUpdate**
> 是否应该更新组件渲染，默认行为为true，当return false时，则不更新。首次渲染和forceUpdate()函数不调用此生命周期函数
- **render**
> 渲染组件
- getSnapshotBeforeUpdate
> 在组件渲染输出之前调用
- **componentDidUpdate**
已废弃
- ~~componentWillUpdate~~
- ~~componentWillReceivePropa~~

### Unmounting
- **componentWillUnmount**
> 在组件卸载及销毁之前直接调用
