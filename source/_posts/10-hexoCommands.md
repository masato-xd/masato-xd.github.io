---
title: HEXO常用命令
date: 2017-07-28 23:10:55
tags: hexo
categories: hexo
---

脑子不用也会生锈，好记性不如烂笔头

<!-- more -->

#### 安装
> `$ npm install -g hexo-cli`

#### 建站
> `$ hexo init "site"` # 创建站点
> `$ cd "site"`
> `$ npm install` # 初始化

#### 文章
> `hexo new "新的文章"`  # 创建新的文章
> `hexo generate == hexo g` # 生成静态文件
> `hexo publish == hexo p` # 发表草稿
> `hexo deploy == hexo d` # 部署
> `hexo g -d == hexo d -g` # 效果一样, 发布+部署

#### 服务器
> `hexo server == hexo s` # 启动服务预览

> 可以通过访问本地端口去预览效果
> `hexo s -debug -p 8888` # 指定端口运行DEBUG模式

#### 清除
清除缓存文件 (db.json) 和已生成的静态文件 (public)。
在某些情况（尤其是更换主题后），如果发现对站点的更改无论如何也不生效
> `hexo clean`

#### 列出信息
> 列出网站内容
> `hexo list <type>`
> `types: page, post, route, tag, category`