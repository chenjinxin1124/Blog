---
title: "k8s强制删除pod&pv&pvc和ns&namespace方法"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "k8s强制删除pod&pv&pvc和ns&namespace方法"    # 文章描述信息
lastmod: 2020-08-17        # 文章修改日期
date: 2022-08-17T11:03:18+08:00
tags : [                    # 文章所属标签
    "K8S",
    "云原生"
]
categories : [              # 文章所属标签
    "K8S",
    "云原生"
]
keywords : [                # 文章关键词
    "K8S",
    "云原生"
]

next: http://changxiangyu.cn/2022/08/%E4%BF%AE%E6%94%B9%E5%8D%9A%E5%AE%A2%E4%B8%BB%E9%A2%98/      # 下一篇博客地址
prev: http://changxiangyu.cn/2020/08/%E5%9C%A8-ubuntu20.04-%E5%BB%BA%E8%AE%BE-openvpn/  # 上一篇博客地址
---
转载: https://www.cnblogs.com/fengdejiyixx/p/15186831.html

注意：以下操作方法十分危险，三思而行！！！

如果名称空间、pod、pv、pvc全部处于“Terminating”状态时，此时的该名称空间下的所有控制器都已经被删除了，之所以出现pod、pvc、pv、ns无法删除，那是因为kubelet 阻塞，有其他的资源在使用该namespace，比如CRD等，尝试重启kubelet，再删除该namespace 也不好使。

正确的删除方法：删除pod--> 删除pvc ---> 删除pv --> 删除名称空间
# 1. 强制删除pod
```
kubectl delete pod <your-pod-name> -n <name-space> --force --grace-period=0
```
解决方法：加参数 --force --grace-period=0，grace-period表示过渡存活期，默认30s，在删除POD之前允许POD慢慢终止其上的容器进程，从而优雅退出，0表示立即终止POD

# 2. 强制删除pv、pvc
```
kubectl patch pv xxx -p '{"metadata":{"finalizers":null}}'
kubectl patch pvc xxx -p '{"metadata":{"finalizers":null}}'
```
直接删除 k8s etcd 数据库中的记录！

# 3. 强制删除ns
在尝试以下命令强制删除也不好使：
```
kubectl delete ns <terminating-namespace> --force --grace-period=0
```
解决方法：

1）运行以下命令以查看处于“Terminating”状态的namespace：
```
kubectl get namespaces
```
2）选择一个Terminating namespace，并查看namespace 中的 finalizer。运行以下命令：
```
$ kubectl get namespace <terminating-namespace> -o yaml
```
输出信息如下：
```
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2019-11-20T15:18:06Z"
  deletionTimestamp: "2020-01-16T02:50:02Z"
  name: <terminating-namespace>
  resourceVersion: "3249493"
  selfLink: /api/v1/namespaces/knative-eventing
  uid: f300ea38-c8c2-4653-b432-b66103e412db
spec:
  finalizers:
  - kubernetes
status:
```
3）导出json格式到文件
```
kubectl get namespace <terminating-namespace> -o json >tmp.json
```
4）编辑tmp.josn，删除finalizers 字段的值
```
{
  "apiVersion": "v1",
  "kind": "Namespace",
  "metadata": {
    "creationTimestamp": "2019-11-20T15:18:06Z",
    "deletionTimestamp": "2020-01-16T02:50:02Z",
    "name": "<terminating-namespace>",
    "resourceVersion": "3249493",
    "selfLink": "/api/v1/namespaces/knative-eventing",
    "uid": "f300ea38-c8c2-4653-b432-b66103e412db"
  },
  "spec": {    #从此行开始删除
    "finalizers": []
  },   # 删到此行
  "status": {
    "phase": "Terminating"
  }
}
```
5）开启proxy
```
kubectl proxy
```
执行该命令后，当前终端会被卡住
6）打开新的一个窗口，执行以下命令
```
curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/<terminating-namespace>/finalize
```
输出信息如下：
```
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "istio-system",
    "selfLink": "/api/v1/namespaces/istio-system/finalize",
    "uid": "2e274537-727f-4a8f-ae8c-397473ed619a",
    "resourceVersion": "3249492",
    "creationTimestamp": "2019-11-20T15:18:06Z",
    "deletionTimestamp": "2020-01-16T02:50:02Z"
  },
  "spec": {
    
  },
  "status": {
    "phase": "Terminating"
  }
}
```
7）确认处于Terminating 状态的namespace已经被删除
```
kubectl get namespaces
```
如果还有处于Terminating 状态的namespace，重复以上操作，删除即可！


