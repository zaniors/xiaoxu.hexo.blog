---
title: 如何用vscode调试js代码
date: 2020-03-26
tags: [vscode, debugger]
categories: [前端]
thumbnail: 'https://cdn.compelcode.com/image/fe/debug.jpg'
---

# 在vscode中调试js代码
在vscode中添加文件配置，并且在vscode中打好断点即可调试

> .vscode/launch.json

### 1、直接利用node环境调试js
``` launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debugger For Node",
      "type": "node",
      "request": "launch",
      "program": "${file}"
    }
  ]
}
```

- program: 表示调试的程序绝对路径
 - 当前打开的文件: ${file}
 - 调试指定的文件: ${workspaceRoot}/app.js

### 2、利用vscode中Debugger for Chrome扩展
``` launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debugger For Chrome",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```
- url: 需要调试的程序在本地开的web服务器

![vscode debugger](https://cdn.compelcode.com/image/fe/debugger-for-node.jpg)