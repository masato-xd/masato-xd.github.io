---
title: Hexo添加LiveRe评论系统
date: 2017-06-29 21:53:21
tags: 随笔
categories: 小记
---
> 参照的是这位[大虾的分享](https://blog.smoker.cc/web/add-comments-livere-for-hexo-theme-next.html)添加来必力评论系统，在这里就不重复造轮子了，只提一些注意事项

<!--more -->
----------

`layout/_scripts/third-party/comments/ `目录中添加 `livere.swig`
`layout/_scripts/third-party/comments.swig` 

> third-party、comments、comments.swig、livere.swig 这四个文件需要自己创建

`livere_uid: your uid`
> 配置在`theme/next/_config.xml`中已经有了，不需要重复添加，修改uid即可

`layout/_partials/comments.swig `文件中条件最后追加 LiveRe 插件是否引用的判断逻辑：
> 判断条件已经存在不需要追加

