---
title: 将Next主题备份至GitHub
date: 2017-06-28 03:07:39
tags: 随笔
categories: 小记
---

# 将Next主题备份至GitHub

当Blog使用Hexo+Next的情况下，使用了Next的主题，要将本地源文件push到GitHub，配置是无法上传的，这样更换电脑以后，配置会很麻烦
下面介绍一种方法，分享给大家

----------
#### 首先备份（上传到GitHub）
- 准备前的工作：
 1. username.github.io命名的仓库
 2. 创建两个分支，master和hexo（任意名字）
 3. 本地有hexo源文件
- 将hexo分支设置为默认的分支
> master分支负责将源文件中生成的静态文件显示到页面不用管

- 将本地hexo源文件中的_config.yml，themes/，source/，scaffolds/，package.json，.gitignore复制至username.github.io文件夹
- 将themes/next/主题中的.git文件夹删除
> 不删除将无法push

- 在这个目录中，创建站点，分别使用下面命令(需要确认是不是hexo分支，git branch)
```bash
# mac需要sudo
(sudo)npm install
(sudo)npm install hexo-cli -g
(sudo)npm install hexo-deployer-git
```
- 执行git add .、git commit -m “提交内容描述”、git push origin hexo，提交hexo源文件到GitHub

#### 修改(上传)
#### 其他电脑恢复