---
layout: post
title: Github Page+Jekyll搭建个人博客
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

马上就要离职了，觉得以后的日子绝对有很多值得总结和记录的地方，因此决定搞一个博客，这篇文章也作为这个博客的开山之作。

<!--more-->

## 方案选型
1. VPS + WordPress  
这套方案是最灵活，体验也相对较好的，插件也很多，但是需要交钱啊，成本大概在每年400块吧。

2. Github Page + Hexo/Hugo/Jekyll  
免费，但是有点折腾，需要有一定的编程基础，其中Hexo/Hugo/Jekyll这三个静态网页生成框架也各有不同。  
    * Hexo: 依托NodeJS，需要装一些依赖，目前最低需要Node >= 12
    * Hugo: 需要Go环境，不需要依赖
    * Jekyll: 如果直接用别人的Theme，什么都不需要安装，Github原生支持
  
我个人不喜欢在电脑上安装各种环境，所以选择了Jekyll。


## 注册一个邮箱
不想让已有的人际关系认出我来，我注册了一个新的邮箱。Yandex是不需要电话验证的，而且对大陆用户基本友好。
顺便说一下同样不需要电话验证的Mail邮箱，这个大陆用户是没法注册的，即使你注册了，发现不是Mail的运营区域，还是会封号。


## Github上搭建仓库
这个不多说了


## 自定义主题、VSCode插件、git配置
我使用的[Tale主题](https://github.com/chesterhow/tale/)，当然[jekyllthemes.org](http://jekyllthemes.org/)有更多选择。  
VSCode插件里面有个jekyll-post，在右键里面会有个New post，它会帮你自动添加日期和md头。  
git的remote可以带着密码，如下所示
```
[remote "origin"]
	url = http://username:password@github.com/smallyuu/Blog.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```


## 参考资料
Tale主题：[https://github.com/chesterhow/tale/](https://github.com/chesterhow/tale/)  
Jekyll文档：[https://jekyllrb.com/docs/](https://jekyllrb.com/docs/)