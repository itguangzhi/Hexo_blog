---
title: themes
date: 2018-03-08 10:41:53
tags: themes
---


# hexo模板
[当前使用的-smackdown](https://github.com/smackgg/hexo-theme-smackdown)

>很不错的网站，很符合心意，里面的内容清晰简单，还可以灵活配置相关内容

## 中文介绍
### 一、关于主题
响应式开发
崇尚极致的性能。
主题能很好的兼容移动端。
在 yilia 的基础上做了SEO优化, 添加了关键字和描述。
主题不支持IE6，7，8。以后也不会

### 二、使用细节
安装
```bash
$ git clone https://github.com/smackgg/hexo-theme-smackdown themes/smackdown
```
配置

修改hexo根目录下的 _config.yml 文件

theme: smackdown
更新
```bash
cd themes/yilia
git pull
```
### 三、配置
主题配置文件在主目录下的_config.yml，请根据自己需要修改使用。 完整配置例子，可以参考我的博客备份
```
# Header
menu:
  主页: /
  所有文章: /archives
  # 随笔: /tags/随笔

# SubNav
subnav:
  # github: "#"
  # zhihu: "#"
  # mail: "#"
  # qq: "#"
  # weibo: "#"
  # rss: "#"
  #douban: "#"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"

rss: /atom.xml

# Content
# excerpt_link: 阅读全文
excerpt_link: more
fancybox: true
mathjax: true

# 是否开启动画效果
animate: true

# 是否在新窗口打开链接
open_in_new: false

# 百度统计、谷歌统计
# baidu_tongji: true
# google_analytics: true

favicon: http://7xkj1z.com1.z0.glb.clouddn.com/head.jpg

#你的头像url
avatar: http://7xkj1z.com1.z0.glb.clouddn.com/head.jpg

#是否开启分享
share: true

#是否开启多说评论，填写你在多说申请的项目名称 duoshuo: duoshuo-key
#若使用disqus，请在博客config文件中填写disqus_shortname，并关闭多说评论
# duoshuo: your duoshuo id

#是否开启云标签
tagcloud: true

#是否开启文章阅读量
leancloud_visitors:
  enable: false
  # app_id: your app_id
  # app_key: your app_key


#是否开启友情链接
#不开启——
#friends: false
#开启——
friends:
  smackdown: https://github.com/smackgg/hexo-theme-smackdown
#是否开启“关于我”。
#不开启——
aboutme: false
#开启——
# aboutme:

swift_search:
  enable: true

```
### 四、其它
关于阅读数

想要添加阅读数需要再进行一些操作，需要注册leancloud账号，添加appid和appkey至主题的_config.yml文件中相应位置。

### 是否开启文章阅读量
leancloud_visitors:
  enable: false
  app_id: your_app_id
  app_key: your_app_key
详细过程见：为smackdown添加阅读数

### 多多益善

同理，自己注册多说账号，添加到主题配置文件的相应位置。
