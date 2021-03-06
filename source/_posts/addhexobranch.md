---
title: 将Next+Hexo备份至GitHub
date: 2017-06-28 03:07:39
tags: hexo
categories: hexo
---


当Blog使用Hexo+Next的情况下，使用了Next的主题，要将本地源文件push到GitHub，配置是无法上传的，这样更换电脑以后，配置会很麻烦
下面介绍一种方法，分享给大家
<!-- more -->

----------
#### 首先备份（上传到GitHub）
1. 准备前的工作：
 - username.github.io命名的仓库
 - 创建两个分支，master和hexo（任意名字）
 - 本地有hexo源文件
 - 将hexo分支设置为默认的分支
> master分支负责的是将源文件中生成的静态文件显示到页面，不要修改

2. 将本地hexo源文件中的_config.yml，themes/，source/，scaffolds/，package.json，.gitignore复制至username.github.io文件夹
3. 将themes/next/主题中的.git文件夹删除
> 不删除将无法push

4. 在这个目录中，创建站点，分别使用下面命令(需要确认是不是hexo分支，git branch)
```bash
# mac需要sudo
(sudo)npm install
(sudo)npm install hexo-cli -g
(sudo)npm install hexo-deployer-git
```
5. 执行git add .、git commit -m “提交内容描述”、git push origin hexo，提交hexo源文件到GitHub

#### 修改(上传)
当你在本地修改内容(主题,文章,页面...)后：
1. 分别执行**git add .**、 **git commit -m "描述"**、**git push origin hexo**，将修改内容提交到GitHub
2. 然后在执行**hexo g -d** 将文章发布出去


#### 其他电脑恢复
> 首先本地要安装[node.js](https://nodejs.org/),[git](https://github.com/)

1. 将仓库clone到本地
```bash
git clone git@github.com:username@username.github.io.git
```
2. 安装一遍Hexo咯(进入目录中)
```powershell
npm install 
npm install hexo-cli g
npm install hexo-deployer-git
```
这样就好了，快去试试吧

