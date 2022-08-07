---
title: "推送到GitHub"
author: "陈金鑫"              # 文章作者
description : "推送到GitHub"    # 文章描述信息
date: 2022-08-01            # 文章编写日期
lastmod: 2022-08-01         # 文章修改日期

tags : [                    # 文章所属标签
    "博客",
    "GitHub"
]
categories : [              # 文章所属标签
    "博客",
    "GitHub",
]
keywords : [                # 文章关键词
    "博客",
    "GitHub",
]
---
# 初始化本地仓库
```
git init
```
# 添加代码
```
git add .
```
# 提交代码
```
git commit -m'first commit'
```
#  添加远程仓库地址
```
git remote add origin git@github.com:jinyumantangcjx/hugo.git
```
# 把本地仓库的变化连接到远程仓库主分支
```
git push --set-upstream origin master
```
