---
title: "将博客部署在GitHub上"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "将博客托管在github"    # 文章描述信息
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

next: my-first-post.md    # 下一篇博客地址
prev: 推送到GitHub.md  # 上一篇博客地址
---
# 准备要部署的内容
在github 上创建一个仓库: chenjinxin.github.io

要向仓库中存放的内容，使用 hugo 命令生成的。在当前目录下，运行 hugo 命令：
```
➜  quickstart git:(master) ✗ pwd
/home/cjx/quickstart
➜  quickstart git:(master) hugo 
Start building sites … 
hugo v0.101.0-9f74196ce611cdf6d355bfb99fd8eba5c68ef7f8+extended linux/amd64 BuildDate=2022-06-28T10:02:18Z VendorInfo=snap

                   | EN  
-------------------+-----
  Pages            | 28  
  Paginator pages  |  0  
  Non-page files   |  0  
  Static files     |  0  
  Processed images |  0  
  Aliases          | 12  
  Sitemaps         |  1  
  Cleaned          |  0  

Total in 33 ms
```
执行成功后，会生成一个public 目录，这个目录中的内容，就是我们博客系统的所有内容，我们需要将这些内容存放在Git 仓库中。
# 部署到 GitHub
按照如下步骤将博客内容上传到Git 仓库，在public 目录下，依次执行下面的命令：
```
# 初始化仓库
git init

# 将所有内容添加到git
git add .

# 提交到git 本地
git commit -m "我的博客第一次提交"

# 关联到远程git，注意这里需要写你自己的git 地址
git remote add origin git@github.com:chenjinxin1124/chenjinxin.github.io.git

# 推送到远程git
git push --set-upstream origin master
```