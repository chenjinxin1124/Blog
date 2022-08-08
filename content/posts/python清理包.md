---
title: "python 清理包"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "python 清理包"    # 文章描述信息
lastmod: 2020-08-08         # 文章修改日期
date: 2022-08-08T18:13:18+08:00
tags : [                    # 文章所属标签
    "Python",
    "包管理",
]
categories : [              # 文章所属标签
    "Python",
    "包管理",
]
keywords : [                # 文章关键词
    "Python",
    "包管理",
]

next: http://www.changxiangyu.cn/2022/08/%E5%A9%9A%E7%BA%B1%E7%85%A7-15x15-%E7%9B%B8%E5%86%8C/      # 下一篇博客地址
prev: http://www.changxiangyu.cn/2022/08/%E5%A9%9A%E7%BA%B1%E7%85%A7%E6%91%86%E4%BB%B6/  # 上一篇博客地址
---
pip freeze > test.txt

pip uninstall -r test.txt -y

pip install -r requirements.txt