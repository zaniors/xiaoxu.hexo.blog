---
title: 使用hexo搭建自己的博客
date: 2019-04-19
thumbnail: https://cdn.compelcode.com/image/fe/hexo.jpg
tags: [hexo, nginx, git]
categories: [前端]
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
npm install -g hexo-cli hexo-deployer-git
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
``` bash
$ mkdir /my-git

$ cd /my-git
$ git init --bare zhary-hexo-blog.git

$ vim /my-git/zhary-hexo-blog.git/hooks/post-receive
# 写入以下代码块
#!/bin/bash
git --work-tree=/www/hexo --git-dir=/my-git/zhary-hexo-blog.git checkout -f

chmod -x /my-git/zhary-hexo-blog.git/hooks/post-receive
```

3. 配置Nginx
``` bash
$ mkdir -p /www/hexo    # 作为博客站点的目录
$ vim /etc/nginx/conf.d/hexo.conf   # 新建一份配置

# 写入以下代码块,注释可忽略
server {
    listen       80;
    server_name  .compelcode.com;   # 博客域名
    root         /www/hexo; # 博客的目录

    location / {
    }

    error_page 404 /404/index.html; #   404页面配置
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

### 配置本地hexo
1. 修改_config.yml文件
``` yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: root@compelcode.com:/my-git/zhary-hexo-blog
  branch: master
```

2. 本地部署
``` bash
$ hexo generate --deploy 或者 hexo g -d
```