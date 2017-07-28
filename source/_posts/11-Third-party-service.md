---
title: HEXO开启第三方服务
date: 2017-07-28 23:11:58
tags: hexo
categories: hexo
---

静态站点拥有一定的局限性，因此我们需要借助于第三方服务来扩展站点的功能

<!-- more -->

### 添加不蒜子统计
> “不蒜子”与百度统计谷歌分析等有区别：“不蒜子”可直接将访问次数显示在您在网页上（也可不显示）；对于已经上线一段时间的网站，“不蒜子”允许您初始化首次数据。。
> [不蒜子官网](http://ibruce.info/2015/04/04/busuanzi/)

主题配置文件中
`/hexo-site/themes/next/_config.yml`
```bash
busuanzi_count:
  # count values only if the other configs are false
  enable: true # 默认false
  # custom uv span for the whole site
  site_uv: true # 代表在页面底部显示站点的UV值。
  site_uv_header: <i class="fa fa-user"></i> 访问
  site_uv_footer: 
  # custom pv span for the whole site
  site_pv: true # 代表在页面底部显示站点的PV值。
  site_pv_header: <i class="fa fa-eye"></i> 总访问量
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true # 代表在文章页面的标题下显示该页面的PV值（阅读数）。
  page_pv_header: <i class="fa fa-file-o"></i> 浏览
  page_pv_footer: 次
```
{% note warning %}
注意DEBUG模式下数值会异常，部署就好了
{% endnote %}

----------
### 开启腾讯分析
主题配置文件中
`/hexo-site/themes/next/_config.yml`
```powershell
# Tencent analytics ID
tencent_analytics: yourID
```
主要步骤:
1. 登录[腾讯分析](ta.qq.com)
2. 右上角站点列表
3. 新增站点
4. 获取ID
```powershell
<script type="text/javascript" src="http://tajs.qq.com/stats?sId=1234567" charset="UTF-8"></script>
```


----------


打完，收工