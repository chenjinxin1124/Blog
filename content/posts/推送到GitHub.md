---
title: "推送到GitHub"
date: 2022-08-01T19:52:18+08:00
draft: false
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
