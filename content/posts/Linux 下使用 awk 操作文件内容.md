---
title: "Linux 下使用 awk 操作文件内容"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "Linux 下使用 awk 操作文件内容"    # 文章描述信息
lastmod: 2020-08-17     # 文章修改日期
date: 2022-08-17T10:03:18+08:00
tags : [                    # 文章所属标签
    "Linux",
    "awk"
]
categories : [              # 文章所属标签
    "Linux",
    "awk"
]
keywords : [                # 文章关键词
    "Linux",
    "awk"
]

next: http://changxiangyu.cn/2022/08/%E4%BF%AE%E6%94%B9%E5%8D%9A%E5%AE%A2%E4%B8%BB%E9%A2%98/      # 下一篇博客地址
prev: http://changxiangyu.cn/2020/08/%E5%9C%A8-ubuntu20.04-%E5%BB%BA%E8%AE%BE-openvpn/  # 上一篇博客地址
---
参考: https://blog.csdn.net/laobai1015/article/details/98628860

## 需求
1. 把 /etc/rancher/k3s/k3s.yaml 文件的内容，每行前面加 4 个空格，追加到 ai-service-platform/yamls/config/kube-config.yaml 文件中；
2. 把 ai-service-platform/yamls/config/kube-config.yaml 文件中的 127.0.0.1 替换成 $master_host_ip。

脚本：
```
function fix_kube_config(){
	sudo awk '{print "    " $0 >> "ai-service-platform/yamls/config/kube-config.yaml"}' /etc/rancher/k3s/k3s.yaml
	sed -i "s/127.0.0.1/$master_host_ip/g" ai-service-platform/yamls/config/kube-config.yaml
}
```

## 扩展
在前面、后面、指定列添加相同字符
1. 给一个文件中的每一行开头插入字符的方法：awk '{print "需要添加的字符" $0}' fileName
2. 给一个文件中的每一行结尾插入字符的方法：awk '{print $0 "需要添加的字符"}' fileName
3. 给一个文件中的每一行的指定列插入字符的方法：awk '$0=$0 X"' fileName

删除某一个列
1. 删除文件中的第一列：awk '{$1="";print $0}' fileName 或者另一种方法 sed -e 's/[^ ]* //' text
2. 删除指定列：awk '{$Num="";print $0}' fileName

把Num换成要删除的列数即可

## 练习
给文件中的每一行开头添加drop tables
```
awk '{print "drop table "$0}' aa.txt > bb.txt
```
给文件中的每一行结尾添加分号
```
awk '{print $0";"}' bb.txt > cc.txt
```
