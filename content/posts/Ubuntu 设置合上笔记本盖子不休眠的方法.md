---
title: "Ubuntu 设置合上笔记本盖子不休眠的方法"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "Ubuntu 设置合上笔记本盖子不休眠的方法"    # 文章描述信息
lastmod: 2020-08-13        # 文章修改日期
date: 2022-08-13T15:33:18+08:00
tags : [                    # 文章所属标签
    "Ubuntu",
    "笔记本"
]
categories : [              # 文章所属标签
    "Ubuntu",
    "笔记本"
]
keywords : [                # 文章关键词
    "Ubuntu",
    "笔记本"
]

next: http://changxiangyu.cn/2022/08/%E4%BF%AE%E6%94%B9%E5%8D%9A%E5%AE%A2%E4%B8%BB%E9%A2%98/      # 下一篇博客地址
prev: http://changxiangyu.cn/2020/08/%E5%9C%A8-ubuntu20.04-%E5%BB%BA%E8%AE%BE-openvpn/  # 上一篇博客地址
---
编辑文件：
```
sudo gedit /etc/systemd/logind.conf
```
```
#HandlePowerKey按下电源键后的行为，默认power off
#HandleSleepKey 按下挂起键后的行为，默认suspend
#HandleHibernateKey按下休眠键后的行为，默认hibernate
#HandleLidSwitch合上笔记本盖后的行为，一般为默认suspend（改为ignore；即合盖不休眠）在原文件中，还要去掉前面的#
```

```
然后将其中的：
#HandleLidSwitch=suspend
复制一行到下面，去掉“#”号：
HandleLidSwitch=ignore
```

最后重启服务
```
service systemd-logind restart
```