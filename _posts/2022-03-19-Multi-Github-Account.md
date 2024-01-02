---
layout: post
title: Github多账户ssh管理代码的解决方案
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

之前Github还可以用密码账号的方式推拉代码，但现在仅允许使用ssh方式进行操作，这对于有多个Github账号的人来说很不友好，目前只能写一篇文章记录下操作步骤。


<!--more-->


## 使用已有的私钥

#### 使用 ssh-agent 
```
ssh-add ~/.ssh/xxx_github_rsa

# 修改 ssh-add 私钥
ssh-add -d ~/.ssh/xxx_github_rsa # 先删除旧的私钥
ssh-add ~/.ssh/xxx2_github_rsa # 添加新的私钥

ssh-add -l # 列举所有添加的私钥
ssh-add -D # 删除所有添加的私钥
```

#### 仓库配置 .git/config
```
$ vim .git/config
[core]
    sshCommand = "ssh -i ~/.ssh/xxx_github_rsa"
[remote "origin"]
    url = git@github.com:xxx/Blog.git
```
注意：
1. url里面冒号后面是用户名，也就是ssh的用户名
2. 如果发现push代码时，user不是仓库对应的user，尝试添加对应的user的私钥


## 最佳体验

1. 为每个账户创建公钥私钥
2. 添加每一个私钥 ssh-add ~/.ssh/xxx_github_rsa
3. git clone git@github.com:xxx/....
4. 如果有需要，不常用的仓库配置其.git/config