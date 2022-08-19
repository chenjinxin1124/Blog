---
title: "k3s手动导入镜像"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "k3s手动导入镜像"    # 文章描述信息
lastmod: 2020-08-19        # 文章修改日期
date: 2022-08-19T17:36:18+08:00
tags : [                    # 文章所属标签
    "K3S",
    "云原生"
]
categories : [              # 文章所属标签
    "K3S",
    "云原生"
]
keywords : [                # 文章关键词
    "K3S",
    "云原生"
]

next: http://changxiangyu.cn/2022/08/%E4%BF%AE%E6%94%B9%E5%8D%9A%E5%AE%A2%E4%B8%BB%E9%A2%98/      # 下一篇博客地址
prev: http://changxiangyu.cn/2020/08/%E5%9C%A8-ubuntu20.04-%E5%BB%BA%E8%AE%BE-openvpn/  # 上一篇博客地址
---
准备镜像文件

导入
```
sudo k3s ctr images import pause.tar.gz
```
查看
```
crictl images|grep pause
```