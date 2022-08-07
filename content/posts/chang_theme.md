---
title: "修改博客主题"
author: "陈金鑫"              # 文章作者
description : "修改博客主题"    # 文章描述信息
date: 2022-08-01            # 文章编写日期
lastmod: 2022-08-01         # 文章修改日期

tags : [                    # 文章所属标签
    "博客",
]
categories : [              # 文章所属标签
    "博客",
]
keywords : [                # 文章关键词
    "博客",
]
---
### 下载博客主题
创建好博客项目后，接下来是下载[hugo博客的主题](https://link.zhihu.com/?target=https%3A//themes.gohugo.io/)，这里有很多主题，我们可以任意挑选，比如我们选择了[bootstrap4-blog 主题](https://link.zhihu.com/?target=https%3A//themes.gohugo.io/hugo-theme-bootstrap4-blog/)。

然后在 Blog 目录下使用git 命令来下载主题：
```
git clone https://github.com/alanorth/hugo-theme-bootstrap4-blog.git themes/hugo-theme-bootstrap4-blog
```
下载下来的主题会放在themes 目录中：
```
└── hugo-theme-bootstrap4-blog
    ├── CHANGELOG.md
    ├── LICENSE.txt
    ├── README.md
    ├── archetypes
    ├── assets
    ├── exampleSite         # 本主题示例内容
    |      ├── content      # 示例博客文章
    │      |-- static
    │      |-- config.toml  # 本主题配置
    ├── i18n
    ├── images
    ├── layouts
    ├── package-lock.json
    ├── package.json
    ├── screenshot.png
    ├── source
    ├── theme.toml      
    └── webpack.config.js
```
### 使用主题

```
vim config.toml

theme = "hugo-theme-bootstrap4-blog"
```