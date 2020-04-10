---
title: Typescript之泛型使用场景
date: 2019-07-12
tags: [typescript]
categories: [前端]
thumbnail:
---

我们都知道前端这几年变化很大，以往的用html,css,js三剑客就能编写出我们需要的页面，而现在前端的业务越来越复杂，必须趋向于前端工程化这种软件工程思想。有需求就必然会有产物，对于JS来说，它是一种解释器语言，也就是说运行时才会知道错误类如：使用了未定义的变量、期望返回字符串，却返回了布尔值等，这时TypeScript应运而生，作为JS的超集，具有类型、泛型、枚举、函数重载、命名空间和模块等，让前端如虎添翼！

>这篇主要说说泛型及基本场景

#### 什么是泛型：泛也就是广泛的意思，可以想象成任何类型（不同于any类型），有什么作用呢？可以让某个函数在未来引入其它未知的数据类型
A开发者：开发了一个http请求库
比如以下伪代码：
``` ts
  // http-client.ts
  class HttpClient {
    get(url: string, options?: { header?: any; type?: 'json' }) {
      return new Promise((resolve, reject) => {
        // 为了方便直接返回后端的请求结果
        if (200) {
          resolve({ code: '200', msg: '登录成功', data: { token: 'ue893ADIo3jadai88sweaa' } })
        } else if (401) {
          reject({ code: '401', msg: '登录失败,请重新登录!' })
        }
      })
    }
  }
```

B调用者：
``` ts
  // index.ts
  const httpClient = new HttpClient()
  httpClient.get('/xxx', {}).then(res => console.log(res))
```

> 这段伪代码可以通过TS解决的几处问题

1. B 调用者不清楚后端返回的数据类型
2. A 开发者不可能只固定返回某种数据
3. 返回数据结构公共部分是否可以复用

#### 与后端定义数据接口
先看看第1点问题，调用者无法得知res(这里res的类型是未知的)的数据结构，也没有接口提示
``` ts
  interface IModel {
    code: string;
    msg: string;
    data: IAuthData;
  }

  interface IAuthData {
    token: string;
  }

  const httpClient = new HttpClient()

  httpClient.get('/xxx', {}).then((res: IModel) => console.log(res))
```
很简单，通过TS的interface定义和后端约定的数据接口

#### 通过泛型接口支持未来的接口
上面A 开发者返回一段数据始终是any类型，这样会丢失数据信息，数据可能是A数据类型，可能是B数据类型，取决于调用者请求的后端URL。
``` ts
  // 定义泛型函数
  class HttpClient {
    // 前面T表示申明泛型
    // 后面T表示返回Promise泛型
    get<T>(url: string, options?: { header?: any; type?: 'json' }): Promise<T> {
      return new Promise((resolve, reject) => {
        // 为了方便直接返回后端的请求结果
        if (200) {
          resolve({ code: '200', msg: '登录成功', data: { token: 'ue893ADIo3jadai88sweaa' } } as any)
        } else if (401) {
          reject({ code: '401', msg: '登录失败,请重新登录!' })
        }
      })
    }
  }
```
B 调用者：使用泛型接口复用接口。IResponse就是一个泛型接口，将code和msg复用。<IResponse<IAuthData>>，IAuthData接口作为IResponse的参数传入
``` ts
  interface IAuthData {
    token: string;
  }

  interface IUserData {
    id: number;
    name: string;
    phone: string;
  }

  interface IResponse<T> {
    code: string;
    msg: string;
    data: T;
  }

  const httpClient = new HttpClient()

  httpClient.get<IResponse<IAuthData>>('/login', {}).then(res => console.log(res))
  httpClient.get<IResponse<IUserData>>('/user', {}).then(res => console.log(res))
```

总结：
1. 接口重用性
2. 对于调用者由自己的数据类型来使用函数，对于开发者则支持未来的接口

- app.quicktype.io：[可以将JSON生成数据模型，支持TS](https://app.quicktype.io/)