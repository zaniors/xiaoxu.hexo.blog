---
title: react-lifecycle
date: 2019-04-29 09:48:40
tags: [react]
categories: [前端]
thumbnail: http://cdn.compelcode.com/image/fe/react-lifecycle.png
---

不管学习任何框架，掌握框架的生命周期函数，才知道组件在某一时刻在执行什么任务

## React(v16.3)的生命周期主要分为三大块
![react-lifecycle-v16.3](image/fe/react-lifecycle-v16.3.png)

### Mounting
- **constructor**
- static getDerivedStateFromProps
- **render**
- **componentDidMount**
已废弃
- ~~componentWillMount~~ 

### Update
- static getDerivedStateFromProps
- **shouldComponentUpdate**
- **render**
- getSnapshotBeforeUpdate
- **componentDidUpdate**
已废弃
- ~~componentWillUpdate~~
- ~~componentWillReceivePropa~~

### Unmounting
- **componentWillUnmount**