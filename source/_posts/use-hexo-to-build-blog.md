---
title: 使用hexo搭建自己的博客
date: 2019-04-19 15:19:43
thumbnail: http://cdn.compelcode.com/image/fe/hexo.jpg
tags: 
    - hexo
    - nginx, 
    - git
---

记录使用[Hexo](https://hexo.io/)搭建个人博客，利用Git Hooks部署到云服务器

## 技术点

### 本地搭建hexo(Mac开发环境)
1. 安装[Homebrew](https://brew.sh/index_zh-cn)
``` bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
2. 安装[Node.js](http://nodejs.cn/download/)
``` bash
$ brew install node
```
3. 安装[hexo](https://hexo.io/zh-cn/docs/index.html)
``` bash
npm install -g hexo-cli
```
4. 初始化hexo项目
``` bash
hexo init my-hexo-blog
```
> 提示：Windows安装类似，直接下载node安装包exe文件，最后安装hexo

### 配置云服务器(Centos7,root用户)
1. 安装Git Nginx
可以直接使用centos的yum安装，或者使用源码安装
``` bash
$ yum install -y git nginx
```
2. 配置Git
``` bash 初始化远端git
$ mkdir /my-git

$ cd /my-git
$ git init --bare zhary-hexo-blog.git
```
3.配置Nginx
4.

### 配置本地hexo
``` yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: root@compelcode.com:/my-git/zhary-hexo-blog
  branch: master
```